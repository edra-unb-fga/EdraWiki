# Inicialização desacoplada (resolve timeout do gz_bridge)

Procedimento de inicialização em **dois estágios** para a simulação. Esta abordagem
resolve os problemas de *timeout* no `gz_bridge`, carregando o mundo 3D de forma
independente **antes** de injetar (spawn) o modelo e iniciar o controlador de voo.

!!! info "Contexto da configuração"
    Os comandos abaixo configuram o modelo `x500_mono_cam_down` para inicializar na
    posição `Z = 0.60m`, evitando colisões físicas do gimbal da câmera com o plano do chão.

---

## Terminal 1 — Preparação e mundo base

O primeiro terminal é dedicado exclusivamente a garantir um ambiente limpo e abrir o
motor gráfico.

**1. Limpeza de processos residuais**

```bash
killall -9 px4 gz ruby
```

Encerra execuções anteriores que possam reter portas de comunicação ou uso de memória.

**2. Configuração de caminhos do ambiente (resources)**

```bash
export GZ_SIM_RESOURCE_PATH=~/PX4-Autopilot/Tools/simulation/gz/models:~/PX4-Autopilot/Tools/simulation/gz/worlds
```

**3. Execução do mundo vazio (SITL default)**

```bash
gz sim -v 4 -r default.sdf
```

!!! note
    Aguarde a interface do simulador carregar completamente (plano cinza e céu) antes de
    avançar para a próxima etapa.

---

## Terminal 2 — Injeção do drone (spawn) e PX4

Com o servidor do Gazebo ativo no Terminal 1, o segundo terminal compila as diretrizes
do modelo e faz a conexão da ponte de simulação (SITL).

**1. Acesso ao repositório base**

```bash
cd ~/PX4-Autopilot
```

**2. Inicialização do Autopilot com posição e modelo customizados**

```bash
PX4_GZ_MODEL_POSE="0,0,0.60,0,0,0" make px4_sitl gz_x500_mono_cam_down
```

!!! tip "Acompanhamento da inicialização"
    Assim que executado, o drone aparecerá flutuando no simulador a 60 cm de altura. No
    Terminal 2, os módulos do PX4 realizarão a inicialização do EKF2 (filtro de Kalman
    estendido). O sistema estará pronto quando a mensagem `INFO [commander] Ready for
    takeoff!` for exibida.

---

## Por que isso funciona

No fluxo padrão (`make px4_sitl ...` sozinho), o PX4 tenta subir o Gazebo e injetar o
modelo ao mesmo tempo. Em máquinas mais lentas ou com a arena pesada, o `gz_bridge`
atinge o *timeout* esperando o mundo ficar pronto e a inicialização falha.

Separando em dois estágios — **primeiro** o mundo carrega por completo, **depois** o
drone é injetado — o bridge encontra um mundo já ativo e conecta na hora, sem corrida
de inicialização.
