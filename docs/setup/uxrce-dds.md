# Ponte PX4 ↔ ROS 2 (Micro XRCE-DDS Agent)

O **Micro XRCE-DDS Agent** é a ponte que conecta o cliente uXRCE-DDS embarcado no PX4 ao
ecossistema ROS 2. Sem o agente rodando, os tópicos do drone (`/fmu/in/*`, `/fmu/out/*`)
**não aparecem** no ROS 2 e nenhum nó consegue comandar ou monitorar o veículo.

!!! info "Fonte"
    Baseado em [uXRCE-configuration](https://github.com/edra-unb-fga/uXRCE-configuration).
    Referência oficial: [PX4 uXRCE-DDS Middleware](https://docs.px4.io/main/en/middleware/uxrce_dds.html).

!!! abstract "Você provavelmente já fez isso"
    A instalação do agente já está incluída em
    [Instalação nativa](instalacao-nativa.md#13-agente-micro-xrce-dds-a-ponte-px4-ros-2)
    e em [Docker no Windows](docker-windows.md). Esta página é **referência** — volte
    aqui se precisar reinstalar, entender melhor a ponte, ou configurar com hardware
    real (serial). Não é um passo extra a fazer depois desses guias.

---

## Instalação (standalone, a partir do código-fonte)

No Ubuntu, compile e instale o agente diretamente:

```bash
git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
cd Micro-XRCE-DDS-Agent
mkdir build && cd build
cmake ..
make
sudo make install
sudo ldconfig /usr/local/lib/
```

Isso já busca as dependências necessárias (como o FastCDR).

---

## Instalação alternativa (dentro de um workspace ROS 2)

```bash
# criar o workspace do agente
mkdir -p ~/px4_ros_uxrce_dds_ws/src
cd ~/px4_ros_uxrce_dds_ws/src
git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git

# compilar com colcon
cd ~/px4_ros_uxrce_dds_ws
source /opt/ros/humble/setup.bash
colcon build
```

Para executar a partir do workspace:

```bash
source /opt/ros/humble/setup.bash
source install/local_setup.bash
MicroXRCEAgent udp4 -p 8888
```

---

## Iniciar o agente

### Com o simulador (SITL)

O cliente uXRCE-DDS do simulador PX4 roda sobre **UDP na porta 8888**:

```bash
MicroXRCEAgent udp4 -p 8888
```

!!! tip
    Deixe o agente rodando em um terminal dedicado durante toda a simulação. Confirme a
    ponte com:
    ```bash
    ros2 topic list | grep fmu
    ```

### Com hardware real (Raspberry Pi via serial)

Ao usar hardware, a conexão costuma ser por porta serial (UART). Exemplo usando a UART0
do Raspberry Pi:

```bash
sudo MicroXRCEAgent serial --dev /dev/AMA0 -b 921600
```

O PX4 suporta apenas conexões **UDP** e **serial**.

---

## Problemas comuns

- **`ros2 topic echo` mudo:** o PX4 publica em *best-effort*. Use
  `ros2 topic echo <topico> --qos-reliability best_effort`. Veja
  [modo Offboard](../controle/offboard.md#topicos-essenciais).
- **Conflito de porta 8888 / erro de payload RTPS:** outro processo pode estar na porta.
  Verifique com `sudo lsof -i:8888`. Veja a
  [lista de erros comuns](../simulacao/erros-comuns.md#micro-xrce-dds-agent).
