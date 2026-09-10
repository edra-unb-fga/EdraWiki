# Documentação — Simulação CBR2025 (Arena + Missão 1 de detecção/pouso)

Guia completo de como montar, rodar e depurar o ambiente de simulação da CBR2025:
arena customizada no Gazebo + missão de detecção e pouso nas bases (PX4 SITL + ROS 2).

Inclui o passo a passo de instalação/execução e **todos os problemas encontrados
com suas respectivas soluções** (a versão do código estava desatualizada e teve
vários pontos a corrigir).

---

## 1. Ambiente / versões usadas

Estas foram as versões testadas nesta máquina. Vale registrar porque **vários
problemas vieram justamente de incompatibilidade de versão**.

| Componente        | Versão                    |
|-------------------|---------------------------|
| SO                | Ubuntu 22.04 (kernel 6.8) |
| ROS 2             | Humble                    |
| Gazebo Sim        | 8.11.0 (Harmonic)         |
| PX4-Autopilot     | v1.16.0                   |
| Micro XRCE-DDS Agent | instalado em `/usr/local/bin/MicroXRCEAgent` |
| Python            | 3.10                      |
| protobuf (pip)    | 4.25.9 (fonte de conflito, ver Problema 2) |

Repositórios:
- **Arena:** `https://github.com/edra-unb-fga/Arenas_Gazebo` — branch **`CBR2025`**
- **Missão:** `https://github.com/edra-unb-fga/CBR_ws` — branch **`#CO-Missao-1-bases`**
  (atenção: o nome real da branch começa com `#` e tem um traço só)

---

## 2. Pré-requisitos (assumidos já instalados)

Este guia parte de uma máquina que já tem:

- **PX4-Autopilot** clonado em `~/PX4-Autopilot` e compilando (`make px4_sitl`).
- **Gazebo Harmonic (gz-sim 8)** instalado.
- **ROS 2 Humble** instalado (`/opt/ros/humble`).
- **Micro XRCE-DDS Agent** instalado (a ponte entre PX4 e ROS 2).
- **colcon** (`sudo apt install python3-colcon-common-extensions`).
- Dependências Python da missão: `ultralytics` (YOLO), `opencv-python`, `numpy`,
  `pymap3d`, `simple-pid`, e os bindings `python3-gz-msgs10` / `python3-gz-transport13`.

Se estiver montando do zero, siga primeiro o tutorial oficial do PX4 para SITL + gz,
e depois volte para cá.

---

## 3. Parte A — Instalar a Arena (Arenas_Gazebo / CBR2025)

A arena substitui os modelos/mundos padrão do PX4 pelos da CBR2025
(plataforma, bases de pouso, kits, etc).

### A.1 Clonar e entrar na branch

```bash
cd ~
git clone https://github.com/edra-unb-fga/Arenas_Gazebo.git
cd Arenas_Gazebo
git checkout CBR2025
```

### A.2 Substituir a pasta `gz` do PX4

O PX4 usa os modelos/mundos que estão em `~/PX4-Autopilot/Tools/simulation/gz`.
A arena substitui o **conteúdo** dessa pasta pelo do repositório.

```bash
cd ~/PX4-Autopilot/Tools/simulation
rm -rf gz
cp -r ~/Arenas_Gazebo gz
```

> **Atenção à estrutura:** depois do `cp`, confira que `worlds/` e `models/`
> aparecem **direto** dentro de `gz/`, e não aninhados em `gz/gz/`:
> ```bash
> ls -la ~/PX4-Autopilot/Tools/simulation/gz
> ```

### A.3 Limpar build e subir

```bash
cd ~/PX4-Autopilot
rm -rf build/
make px4_sitl gz_x500_mono_cam_down
```

> Se o Gazebo abrir e você ver a plataforma com as bases de pouso e os kits,
> a arena está OK. **Antes de continuar, leia a seção de Problemas** —
> alguns fixes precisam ser aplicados aqui (magnetômetro, `gz_env.sh`).

---

## 4. Parte B — Instalar a Missão (CBR_ws / detecção + pouso)

### B.1 Clonar e entrar na branch

```bash
cd ~
git clone https://github.com/edra-unb-fga/CBR_ws.git
cd CBR_ws
git checkout '#CO-Missao-1-bases'
```

(As aspas são necessárias por causa do `#`.)

### B.2 Compilar o workspace

```bash
cd ~/CBR_ws
source /opt/ros/humble/setup.bash
colcon build --symlink-install
```

> **O `px4_msgs` demora ~15-18 min** na primeira compilação — ele gera código C++
> para todas as mensagens do PX4. É custo **único**; nas próximas vezes só os
> pacotes Python recompilam (segundos). Ver Problema 1 se o `drone_config` falhar.

### B.3 Rodar a missão (3 terminais)

**Terminal 1 — Simulação PX4 + Gazebo (arena):**
```bash
cd ~/PX4-Autopilot
make px4_sitl gz_x500_mono_cam_down
```

**Terminal 2 — Ponte ROS 2 ↔ PX4:**
```bash
MicroXRCEAgent udp4 -p 8888
```

**Terminal 3 — QGroundControl** (recomendado — ver Problema 8):
```bash
~/Downloads/QGroundControl-x86_64.AppImage
```

**Terminal 4 — A missão:**
```bash
cd ~/CBR_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python ros2 run Missao_1 main
```

> O prefixo `PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python` é obrigatório
> (ver Problema 2).

---

## 5. Problemas encontrados e soluções

Ordem aproximada em que apareceram ao subir a missão.

### Problema 1 — `drone_config` não compila: `__init__.py` faltando

**Sintoma (no `colcon build`):**
```
error: package directory 'drone_config/commander' does not exist
Failed <<< drone_config
```

**Causa:** faltava o arquivo `src/drone_config/drone_config/__init__.py` no repositório
(não estava versionado). Sem ele, o setuptools não reconhece o pacote Python.

**Solução:** criar o arquivo (vazio, como os outros `__init__.py` do projeto) e recompilar:
```bash
touch ~/CBR_ws/src/drone_config/drone_config/__init__.py
cd ~/CBR_ws && colcon build --symlink-install
```

---

### Problema 2 — Conflito de versão do protobuf ao iniciar a missão

**Sintoma:**
```
TypeError: Descriptors cannot be created directly.
If this call came from a _pb2.py file, your generated code is out of date ...
(from gz.msgs10.image_pb2 import Image)
```

**Causa:** a versão do `protobuf` instalada via pip (4.25.9) é mais nova que a usada
para gerar os bindings do Gazebo (`python3-gz-msgs10`). Eles brigam ao importar a
mensagem de imagem da câmera.

**Solução:** rodar a missão forçando a implementação pura-Python do protobuf:
```bash
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python ros2 run Missao_1 main
```
(Alternativa permanente: fixar `protobuf` em 3.20.x no ambiente — mas o prefixo
acima é o menos invasivo.)

---

### Problema 3 — `gz_env.sh` faltando: Gazebo não acha o mundo

**Sintoma:**
```
[Err] File [/default.sdf] resolved to path [/default.sdf] but the path does not exist
[Err] Failed to find world [/default.sdf]
INFO [init] Waiting for Gazebo world... (repete pra sempre)
```

**Causa:** o PX4 gera, durante o build, o arquivo
`build/px4_sitl_default/rootfs/gz_env.sh`, que define `PX4_GZ_WORLDS`,
`PX4_GZ_MODELS` e `GZ_SIM_RESOURCE_PATH`. Ele sumiu/não foi gerado, então o PX4
tentou abrir `/default.sdf` (caminho vazio).

**Solução:** recriar o arquivo apontando para as pastas corretas:
```bash
cat > ~/PX4-Autopilot/build/px4_sitl_default/rootfs/gz_env.sh << 'EOF'
#!/usr/bin/env bash
export PX4_GZ_MODELS=/home/luana/PX4-Autopilot/Tools/simulation/gz/models
export PX4_GZ_WORLDS=/home/luana/PX4-Autopilot/Tools/simulation/gz/worlds
export PX4_GZ_PLUGINS=/home/luana/PX4-Autopilot/build/px4_sitl_default/src/modules/simulation/gz_plugins
export PX4_GZ_SERVER_CONFIG=/home/luana/PX4-Autopilot/src/modules/simulation/gz_bridge/server.config
export GZ_SIM_RESOURCE_PATH=$GZ_SIM_RESOURCE_PATH:$PX4_GZ_MODELS:$PX4_GZ_WORLDS
export GZ_SIM_SYSTEM_PLUGIN_PATH=$GZ_SIM_SYSTEM_PLUGIN_PATH:$PX4_GZ_PLUGINS
export GZ_SIM_SERVER_CONFIG_PATH=$PX4_GZ_SERVER_CONFIG
EOF
```
(Ajuste os caminhos se seu usuário não for `luana`. Um `rm -rf build && make px4_sitl`
também regenera o arquivo.)

---

### Problema 4 — "Found 0 compass": magnetômetro sumiu do modelo do drone

**Sintoma (no shell `pxh>` do PX4):**
```
WARN [health_and_arming_checks] Preflight Fail: no heading reference
WARN [health_and_arming_checks] Preflight Fail: Found 0 compass (required: 1)
```

**Causa (a de verdade):** ao substituir a pasta `gz` pela da arena, o arquivo
`models/x500_base/model.sdf` da arena **sobrescreveu** o original do PX4 e, nesse
processo, o **sensor de magnetômetro foi removido** do modelo. Sem magnetômetro,
o PX4 não arma.

> O README da arena sugere o paliativo `sensor_mag_sim start` no `pxh>`. Isso às vezes
> funciona, mas é **instável**: esse módulo simulado depende do EKF2 já ter posição
> global válida, o que cria uma dependência circular (compass depende de GPS que
> depende de compass). Por isso ele resolvia em algumas execuções e em outras não.

**Solução definitiva:** devolver o sensor de magnetômetro ao `model.sdf`, exatamente
como no modelo original do PX4. No arquivo
`~/PX4-Autopilot/Tools/simulation/gz/models/x500_base/model.sdf` (e no equivalente
em `~/Arenas_Gazebo/gz/models/x500_base/model.sdf`), inserir o bloco abaixo **entre**
o sensor `air_pressure_sensor` e o `imu_sensor`:

```xml
      <sensor name="magnetometer_sensor" type="magnetometer">
        <always_on>1</always_on>
        <update_rate>100</update_rate>
        <magnetometer>
          <x><noise type="gaussian"><stddev>0.0001</stddev></noise></x>
          <y><noise type="gaussian"><stddev>0.0001</stddev></noise></y>
          <z><noise type="gaussian"><stddev>0.0001</stddev></noise></z>
        </magnetometer>
      </sensor>
```

Com o sensor nativo de volta, **não é preciso** o `sensor_mag_sim start`.

---

### Problema 5 — Flags `position`/`velocity` invertidas (offboard não controla)

**Arquivo:** `src/drone_config/drone_config/commander/px4_commander.py`,
método `publish_offboard_control_heartbeat_signal()`.

**Sintoma:** o drone arma mas não decola; a árvore de comportamento marca tudo como
sucesso (`✓`) mesmo o drone parado (porque os nós não verificam o estado real).

**Causa:** o heartbeat do modo offboard anunciava controle por **velocidade**
(`velocity=True`, `position=False`), mas todo o resto do código envia **setpoints de
posição** (`publish_setpoint`). O próprio comentário no código dizia que `position`
"deve ser True". O PX4 recebia setpoints de posição com velocidade `NaN` → ignorava.

**Solução:**
```python
# de:
msg.position = False
msg.velocity = True
# para:
msg.position = True
msg.velocity = False
```

---

### Problema 6 — Tópico `vehicle_status` errado (PX4 v1.16 usa versionamento)

**Arquivo:** `src/drone_config/drone_config/commander/px4_commander.py`.

**Sintoma:** o callback de status nunca dispara; `arming_state`/`nav_state` ficam
sempre `None`; o commander "voa cego" sobre o estado do drone.

**Causa:** a partir do **PX4 v1.16**, mensagens versionadas usam o sufixo `_v<versão>`
no nome do tópico. O `VehicleStatus` tem `MESSAGE_VERSION = 1`, então o tópico real
é `/fmu/out/vehicle_status_v1` — o código antigo assinava `/fmu/out/vehicle_status`
(que não existe mais). Já o `VehicleLocalPosition` tem `MESSAGE_VERSION = 0`,
por isso a posição funcionava (fica sem sufixo).

**Solução:**
```python
# de:
'/fmu/out/vehicle_status'
# para:
'/fmu/out/vehicle_status_v1'
```

---

### Problema 7 — QoS incompatível nas subscriptions (recebe silêncio)

**Arquivo:** `src/drone_config/drone_config/commander/px4_commander.py`.

**Sintoma:**
```
WARN New publisher discovered on topic '/fmu/out/vehicle_local_position',
offering incompatible QoS. No messages will be received from it.
Last incompatible policy: RELIABILITY
```

**Causa:** o PX4 publica os tópicos `/fmu/out/*` com QoS **BEST_EFFORT + TRANSIENT_LOCAL**.
O commander assinava com o QoS padrão (**RELIABLE**), que é incompatível → nenhuma
mensagem chega (posição, status, etc).

**Solução:** criar um perfil de QoS compatível e usá-lo em **todas** as subscriptions
de `/fmu/out/*`:
```python
from rclpy.qos import QoSProfile, ReliabilityPolicy, DurabilityPolicy

self.qos_profile_px4_out = QoSProfile(
    depth=10,
    reliability=ReliabilityPolicy.BEST_EFFORT,
    durability=DurabilityPolicy.TRANSIENT_LOCAL,
)
# usar self.qos_profile_px4_out em vehicle_status_v1, vehicle_local_position,
# vehicle_global_position, etc.
```

---

### Problema 8 — Arm negado: "Resolve system health failures first"

Esse foi o mais difícil e teve **duas causas combinadas**.

**Sintoma:** o comando de arm é enviado mas o PX4 nega repetidamente:
```
WARN [commander] Arming denied: Resolve system health failures first
```
No QGroundControl aparece a mesma mensagem em vermelho.

**Causa 8a — árvore arma cedo demais / sem confirmar.**
O nó `Arm` (`src/common_nodes/common_nodes/flight/arm.py`):
- Enviava o comando de arm ~0.5 s após iniciar, antes do **EKF2 convergir** (posição
  local e global válidas). No momento do arm o drone ainda está em `AUTO_LOITER`,
  modo que **exige posição local E global (GPS) válidas**.
- Mandava o comando **uma única vez** e assumia sucesso, sem checar se armou de fato.
  Se caísse numa janela ruim de inicialização, nunca mais tentava.

**Solução 8a:** (1) esperar a posição ficar válida antes de armar; (2) reenviar o
comando periodicamente até o PX4 confirmar `arming_state == ARMED` (=2).
Para isso, o `px4_commander.py` passou a expor `local_position_valid` e
`global_position_valid` (lidos de `xy_valid/z_valid` e `lat_lon_valid/alt_valid`,
assinando também `/fmu/out/vehicle_global_position`), e o `arm.py` virou:

```python
ARMING_STATE_ARMED = 2
RETRY_INTERVAL_S = 2.0

def update(self):
    self.cmdr.publish_offboard_control_heartbeat_signal()

    if self.cmdr.arming_state == ARMING_STATE_ARMED:
        return py_trees.common.Status.SUCCESS          # confirmado de verdade

    elapsed = time.time() - self.start_time
    position_ready = (self.cmdr.local_position_valid
                      and self.cmdr.global_position_valid) or elapsed > 15.0

    should_retry = (self.last_attempt_time is None
                    or (time.time() - self.last_attempt_time) >= RETRY_INTERVAL_S)

    if self.cmdr.offboard_setpoint_counter >= 10 and position_ready and should_retry:
        self.cmdr.send_command(VehicleCommand.VEHICLE_CMD_COMPONENT_ARM_DISARM, 1.0)
        self.last_attempt_time = time.time()

    return py_trees.common.Status.RUNNING
```

**Causa 8b — sem conexão com estação de solo (GCS).**
Uma das checagens de saúde é `No connection to the ground control station`. Sem uma
GCS conectada, o arm fica intermitente.

**Solução 8b:** **manter o QGroundControl aberto** durante a missão. Com o QGC
conectado, essa checagem passa e o arm fica confiável. (Foi o que fez o drone
finalmente armar e subir de forma consistente.)

---

### Problema 9 — Câmera "congelada" e detecção nunca acha base (gargalo de desempenho)

**Sintoma:** o drone chega a voar, mas a detecção YOLO sempre reporta `no detections`
e a missão nunca pousa numa base.

**Investigação:** salvando os frames crus da câmera, descobrimos que **todos os frames
eram byte-a-byte idênticos** por dezenas de segundos — a câmera estava efetivamente
congelada, entregando sempre a mesma imagem inicial (borrada, do drone no chão).
Medindo o Gazebo:
```
real_time_factor: 0.05  a  0.087   # simulação a ~5-8% da velocidade real
pose do drone: Z ≈ -0.0125         # drone no chão, não subiu
```

**Causa raiz:** a **máquina estava saturada**. Rodando ao mesmo tempo Gazebo server +
janela 3D do Gazebo + QGroundControl + YOLO (inferência de 500–1600 ms por frame na
CPU), o RTF despencou para ~5%. Com o simulador tão lento:
- a câmera renderiza pouquíssimos frames (parece congelada);
- o controle de voo, que espera tempo real, fica instável;
- os temporizadores da missão baseados em `time.time()` (relógio real) expiram antes
  de o drone fazer as coisas em tempo de simulação (ex.: o `Takeoff` espera 5 s de
  relógio e declara "voo estabilizado", mas em 5 s reais passaram só ~0,25 s de
  simulação → o drone mal saiu do chão).

Ou seja: **este é o motivo pelo qual os resultados eram intermitentes** — quando a
máquina estava menos carregada, o RTF melhorava e o drone chegava a subir (~1,6 m).

**Soluções / recomendações (por ordem de impacto):**
1. **Rodar em modo headless** (sem a janela 3D do Gazebo — maior ganho isolado):
   ```bash
   HEADLESS=1 make px4_sitl gz_x500_mono_cam_down
   ```
2. **Rodar o YOLO na GPU** em vez da CPU, se houver placa dedicada (a inferência de
   500–1600 ms/frame indica execução em CPU).
3. **Fechar aplicativos pesados** durante o teste; idealmente rodar numa máquina mais
   forte.
4. **(melhoria de código, do grupo)** fazer a lógica da missão usar **tempo de
   simulação** (`/clock`) em vez de `time.time()`, para ficar robusta a RTF baixo.

> Observação: com o sensor de magnetômetro correto (Problema 4) e o QGC aberto
> (Problema 8b), o drone **arma, entra em OFFBOARD e decola de verdade** (confirmado:
> `Arming: 2`, altitude subindo). O que ainda impede a missão de pousar nas bases é
> o gargalo de desempenho acima — não a lógica da missão em si.

---

## 6. Resumo dos arquivos alterados

**No repositório da missão (`~/CBR_ws`):**
| Arquivo | Alteração |
|---------|-----------|
| `src/drone_config/drone_config/__init__.py` | **criado** (vazio) — Problema 1 |
| `src/drone_config/drone_config/commander/px4_commander.py` | flags offboard, tópico `_v1`, QoS BEST_EFFORT, subscrição de global position + flags de validade — Problemas 5, 6, 7, 8a |
| `src/common_nodes/common_nodes/flight/arm.py` | espera EKF2 + retry até confirmar arm — Problema 8a |

**No ambiente do PX4 (`~/PX4-Autopilot`):**
| Arquivo | Alteração |
|---------|-----------|
| `Tools/simulation/gz/models/x500_base/model.sdf` | readicionado o sensor de magnetômetro — Problema 4 |
| `build/px4_sitl_default/rootfs/gz_env.sh` | recriado (paths do mundo/modelos) — Problema 3 |

**Na arena (`~/Arenas_Gazebo`, branch CBR2025):**
| Arquivo | Alteração |
|---------|-----------|
| `gz/models/x500_base/model.sdf` | readicionado o sensor de magnetômetro — Problema 4 |

> As mudanças no `~/CBR_ws` e no `~/Arenas_Gazebo` são as que valem a pena **commitar**
> nos repositórios do grupo. As mudanças dentro de `~/PX4-Autopilot/build/` são
> geradas/temporárias.

---

## 7. Checklist rápido para rodar

1. [ ] Arena copiada para `~/PX4-Autopilot/Tools/simulation/gz` (worlds/models na raiz)
2. [ ] Magnetômetro presente em `x500_base/model.sdf` (Problema 4)
3. [ ] `CBR_ws` compilado; `drone_config/__init__.py` existe (Problema 1)
4. [ ] Terminal 1: `make px4_sitl gz_x500_mono_cam_down` (ou `HEADLESS=1 ...`)
5. [ ] Terminal 2: `MicroXRCEAgent udp4 -p 8888`
6. [ ] Terminal 3: QGroundControl aberto e conectado (Problema 8b)
7. [ ] Terminal 4: `PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python ros2 run Missao_1 main` (Problema 2)
8. [ ] Conferir o RTF do Gazebo — se estiver muito baixo (<0.3), aliviar a carga (Problema 9)
