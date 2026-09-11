# Modo Offboard: controle por posição × velocidade

O modo **Offboard** transfere a autoridade de controle dos controladores internos do
PX4 para um sistema externo (o *companion computer*, ex: Raspberry Pi), permitindo
navegação autônoma e integração com visão computacional via ROS 2.

!!! info "Material completo"
    Este é um resumo prático. O material aprofundado está em
    [Producao-do-relatorio-da-SAE](https://github.com/edra-unb-fga/Producao-do-relatorio-da-SAE)
    (seção *Estrutura de controle e interação com a PX4*).

---

## Requisitos para operar em Offboard

1. **Fluxo contínuo de comandos** — publique setpoints a no mínimo 2 Hz (recomendado
   ≥ 10 Hz). Se o intervalo passar de 500 ms (`COM_OF_LOSS_T`), o PX4 cai para um modo
   de fallback.
2. **Sequência de inicialização correta:**
   - Começar a enviar setpoints **antes** de mudar para Offboard;
   - Manter o veículo armado antes da transição;
   - Passar nas verificações pré-voo.
3. **Comunicação bidirecional** — além de enviar comandos, monitore o estado do veículo
   (armamento, modo, saúde, posição/velocidade estimadas).

---

## Tópicos essenciais

| Direção | Tópico | Função |
|---------|--------|--------|
| companion → PX4 | `/fmu/in/offboard_control_mode` | Define **qual tipo** de controle será usado |
| companion → PX4 | `/fmu/in/trajectory_setpoint` | Envia os setpoints (posição, velocidade, aceleração) |
| companion → PX4 | `/fmu/in/vehicle_command` | Comandos (armar, mudar modo, etc.) |
| PX4 → companion | `/fmu/out/vehicle_status` | Estado (modo, armamento) |
| PX4 → companion | `/fmu/out/vehicle_local_position` | Posição/velocidade estimadas |

!!! warning "QoS — a pegadinha mais comum"
    O PX4 publica os `/fmu/out/*` com **BEST_EFFORT + TRANSIENT_LOCAL**. Um subscriber
    ROS 2 com o QoS padrão (RELIABLE) é incompatível e **não recebe nada**. Use sempre:
    ```python
    from rclpy.qos import QoSProfile, ReliabilityPolicy, DurabilityPolicy, HistoryPolicy
    qos = QoSProfile(
        reliability=ReliabilityPolicy.BEST_EFFORT,
        durability=DurabilityPolicy.TRANSIENT_LOCAL,
        history=HistoryPolicy.KEEP_LAST, depth=1)
    ```

---

## O `OffboardControlMode` precisa casar com o setpoint

O `OffboardControlMode` anuncia ao PX4 **qual tipo** de setpoint você vai enviar. As
flags **têm que casar** com o que você de fato publica no `trajectory_setpoint`:

=== "Controle por posição"

    ```python
    # anuncia: vou controlar por POSIÇÃO
    msg = OffboardControlMode()
    msg.position = True
    msg.velocity = False
    # ...
    # e envia setpoint de posição (metros, frame NED)
    sp = TrajectorySetpoint()
    sp.position = [10.0, 5.0, -3.0]   # 10 N, 5 L, 3 m acima do solo
    sp.yaw = 0.0
    ```

=== "Controle por velocidade"

    ```python
    # anuncia: vou controlar por VELOCIDADE
    msg = OffboardControlMode()
    msg.position = False
    msg.velocity = True
    # ...
    # e envia setpoint de velocidade (m/s, frame NED)
    sp = TrajectorySetpoint()
    sp.velocity = [1.0, 0.5, 0.0]     # 1 m/s N, 0.5 m/s L, mantém altitude
    sp.yawspeed = 0.0
    ```

!!! danger "Erro clássico (e difícil de achar)"
    Se o `OffboardControlMode` disser `velocity=True` mas você publicar **setpoints de
    posição** (com `velocity = [nan, nan, nan]`), o PX4 não recebe um comando válido e o
    drone fica parado — **mesmo o código não dando erro**. Foi exatamente esse bug que
    travou a Missão 1 da CBR2025. As flags devem sempre corresponder ao setpoint enviado.
    Veja o caso completo em
    [Simulação CBR2025](../simulacao/cbr2025-arena-missao1.md).

---

## Posição × Velocidade — comparação

| Aspecto | Controle por Posição | Controle por Velocidade |
|---------|----------------------|-------------------------|
| Precisão estática | Alta (mantém posição fixa) | Menor — tende a derivar |
| Resposta dinâmica | Mais lenta a mudanças | Mais rápida |
| Complexidade | Mais simples | Exige monitoramento cuidadoso |
| Adaptável a estímulo externo | Menos (segue trajetória fixa) | Mais (responde em tempo real) |
| Integração com visão | Requer converter para posição absoluta | Direta a partir do erro em pixels |

**Quando usar cada um:**

- **Posição** → navegação por waypoints, hover estável, decolagem/pouso preciso, quando
  as posições absolutas são conhecidas de antemão.
- **Velocidade** → seguir alvo em movimento, navegação por visão computacional,
  centralização em cima de uma base, ambientes dinâmicos.

---

## Abordagem híbrida (o que a missão de fato usa)

Missões reais alternam entre os dois conforme a fase. Ex.: **centralizar** em uma base
usa velocidade (ajuste contínuo pelo erro visual em pixels); **navegar** entre bases usa
posição. Exemplo de centralização por velocidade:

```python
def centralizar_alvo(self, alvo_px, centro_img):
    erro_x = centro_img[0] - alvo_px[0]
    erro_y = centro_img[1] - alvo_px[1]
    ganho = 0.005  # pixel -> m/s
    # limita a velocidade para movimentos suaves
    vel_x = max(min(erro_y * ganho, 0.5), -0.5)   # eixo Y da imagem -> N/S
    vel_y = max(min(-erro_x * ganho, 0.5), -0.5)  # eixo X da imagem -> L/O
    self.publish_offboard_control_mode(position=False, velocity=True)
    self.publish_velocity_setpoint(vel_y, vel_x, 0.0)
```

!!! tip "Transição suave entre modos"
    Ao trocar de posição para velocidade (ou vice-versa), atualize o `OffboardControlMode`
    e **comece com velocidade zero** (ou a posição atual) para evitar movimentos súbitos.

---

## Segurança

- **Heartbeat / conectividade:** monitore continuamente a comunicação; detecte perda de link.
- **Limitação de comandos:** restrinja velocidades/acelerações máximas; valide todo setpoint.
- **Failsafe por fase:** tenha comportamentos seguros para cada etapa da missão.
- **Teste progressivo:** primeiro no SITL, depois com o drone preso, depois manobras
  simples em ambiente controlado.
