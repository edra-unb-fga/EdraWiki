# Instalação nativa (Linux)

Passo a passo completo para configurar o ambiente de simulação do drone **direto no
Ubuntu** (sem Docker): PX4 Autopilot, Gazebo Harmonic e ROS 2 Humble.

!!! abstract "Onde este guia se encaixa"
    Este é um dos **dois** caminhos possíveis para montar o ambiente (veja
    [Primeiros passos](../primeiros-passos.md)). Use este guia **se seu sistema é Linux**.
    Se é Windows, use [Docker no Windows](docker-windows.md) em vez deste. Depois de
    terminar aqui, o próximo passo é
    [Simulação CBR2025 — Arena + Missão 1](../simulacao/cbr2025-arena-missao1.md).

!!! info "Ambiente de referência"
    Ubuntu 22.04 · PX4 **v1.16.0** · Gazebo Harmonic (gz-sim 8.11.0) · ROS 2 Humble.
    Manter o PX4 na **v1.16.0** é importante para compatibilidade com os modelos da arena.

Se você está começando do zero (instalar o Ubuntu, configurar Git/SSH), veja antes o
tutorial de onboarding em
[Configuracoes_Basicas_para_Controle](https://github.com/edra-unb-fga/Configuracoes_Basicas_para_Controle).

---

## 1. Instalação passo a passo

### 1.1 ROS 2 Humble

No Ubuntu 22.04, o Humble é a versão LTS recomendada. Depois de instalar o ROS 2,
adicione as dependências de build:

```bash
sudo apt install python3-colcon-common-extensions python3-rosdep python3-vcstool
sudo rosdep init
rosdep update
```

### 1.2 PX4 Autopilot (v1.16.0)

```bash
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
cd ~/PX4-Autopilot
git checkout v1.16.0
bash ./Tools/setup/ubuntu.sh
```

!!! note
    Reinicie o sistema após rodar o `ubuntu.sh`.

### 1.3 Agente Micro-XRCE-DDS (a ponte PX4 ↔ ROS 2)

O PX4 conversa com o ROS 2 pelo protocolo XRCE-DDS. Sem este agente, os tópicos do
drone (sensores, estado, GPS) **não aparecem** no ROS 2.

```bash
# Clonar o repositório do agente
git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
cd Micro-XRCE-DDS-Agent

# Criar pasta de build e compilar
mkdir build && cd build
cmake ..
make
sudo make install

# Atualizar as bibliotecas do sistema
sudo ldconfig /usr/local/lib/
```

!!! warning
    Sem este agente, `ros2 topic list` não mostra nada vindo do firmware PX4.

### 1.4 Workspace ROS 2 (mensagens px4_msgs)

Necessário para o ROS 2 interpretar as mensagens do drone:

```bash
mkdir -p ~/px4_ros_com_ws/src
git clone https://github.com/PX4/px4_msgs.git ~/px4_ros_com_ws/src/px4_msgs
cd ~/px4_ros_com_ws && colcon build
```

### 1.5 QGroundControl (QGC)

É a estação de controle de solo (GCS) usada para monitorar o drone, calibrar
sensores e alterar parâmetros do PX4 em tempo real.

```bash
# 1. Dependências de rede e vídeo
sudo apt install libpulse-dev libqt5gui5 libqt5widgets5 libqt5serialport5 -y

# 2. Permissões de porta serial (IMPORTANTE)
sudo usermod -a -G dialout $USER
sudo apt-get remove modemmanager -y

# 3. Baixar o AppImage oficial
wget https://s3-us-west-2.amazonaws.com/qgroundcontrol/latest/QGroundControl.AppImage

# 4. Permissão de execução e iniciar
chmod +x QGroundControl.AppImage
./QGroundControl.AppImage
```

!!! tip
    Se aparecer "Time Jump Detected" ou dessincronização no ROS 2, feche o QGC para
    reduzir a carga da CPU.

### 1.6 Arena CBR2025

O passo a passo completo de como clonar e configurar a arena da CBR2025 (e os problemas
de sensores específicos dela) está no guia dedicado:
[Simulação CBR2025 — Arena + Missão 1](../simulacao/cbr2025-arena-missao1.md). Ele
assume que os passos 1.1 a 1.5 acima já foram feitos.

---

## 2. Fluxo de execução da simulação

Abra **4 terminais** na ordem abaixo (e deixe o QGroundControl aberto):

=== "Terminal 1 — Micro-XRCE Agent"

    Inicia a ponte de comunicação.
    ```bash
    MicroXRCEAgent udp4 -p 8888
    ```

=== "Terminal 2 — PX4 standalone"

    Inicia o firmware preparando-o para o Gazebo.
    ```bash
    cd ~/PX4-Autopilot
    PX4_GZ_STANDALONE=1 PX4_GZ_WORLD=default make px4_sitl gz_x500
    ```

=== "Terminal 3 — Gazebo (arena)"

    Carrega o mundo 3D da CBR2025.
    ```bash
    GZ_SIM_RESOURCE_PATH=~/Arenas_Gazebo/gz/models:~/PX4-Autopilot/Tools/simulation/gz/models \
      gz sim -r ~/Arenas_Gazebo/gz/worlds/default.sdf
    ```

=== "Terminal 4 — Script de controle (ROS 2)"

    Executa a lógica da missão.
    ```bash
    cd ~/ros2_ws && source install/setup.bash
    ros2 run <seu_pacote> <seu_script>
    ```

!!! tip "Abra o QGroundControl"
    Sem uma GCS conectada, o PX4 recusa o armamento com
    `Preflight Fail: No connection to the ground control station` e o drone nunca decola.

---

## 3. Se algo der errado

Não existe uma lista de problemas aqui — o troubleshooting fica em dois lugares,
dependendo do tipo de erro:

- **Erro genérico** de build, comunicação PX4/ROS2/Gazebo, ou da ponte uXRCE →
  [Lista de erros comuns](../simulacao/erros-comuns.md).
- **Erro específico da arena CBR2025** (sensores do modelo, arm negado, versionamento
  do PX4 v1.16) → [Simulação CBR2025](../simulacao/cbr2025-arena-missao1.md).
- **Timeout do `gz_bridge`** esperando o mundo carregar →
  [Inicialização desacoplada](../simulacao/inicializacao-desacoplada.md).

---

## 4. Referência rápida de comandos

Comandos no console `pxh>` do PX4:

| Comando | Descrição |
|---------|-----------|
| `listener estimator_status 1` | Ver status do EKF2 / magnetômetro |
| `listener vehicle_status 1` | Ver estado do drone |
| `commander arm` / `disarm` | Armar / desarmar |
| `commander takeoff` / `land` | Decolar / pousar |
| `commander mode offboard` | Entrar em modo offboard |
| `ekf2 stop` / `start` | Reiniciar o estimador de posição |

Comandos no Ubuntu:

| Comando | Descrição |
|---------|-----------|
| `pkill -9 -f gz` / `pkill -9 -f px4` | Matar processos para reiniciar limpo |
| `gz topic -l \| grep mag` | Verificar se o magnetômetro publica |
| `MicroXRCEAgent udp4 -p 8888` | Iniciar o agente da ponte |
| `ros2 topic list \| grep fmu` | Ver os tópicos do PX4 no ROS 2 |

---

**Próximo passo:** com o ambiente instalado, siga para
[Simulação CBR2025 — Arena + Missão 1](../simulacao/cbr2025-arena-missao1.md).
