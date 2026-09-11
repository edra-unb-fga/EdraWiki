# Lista de erros comuns

Catálogo de erros já enfrentados pela equipe na configuração do ambiente
(PX4 · ROS 2 · Gazebo), com a solução que funcionou. Baseado no
[gazebo-px4/docs/LISTA_DE_ERROS.md](https://github.com/edra-unb-fga/gazebo-px4/blob/main/docs/LISTA_DE_ERROS.md).

!!! tip
    Para os problemas específicos de sensores/arming da arena CBR2025, veja também
    [Simulação CBR2025](cbr2025-arena-missao1.md). Para o Windows/WSL2, veja
    [Docker no Windows](../setup/docker-windows.md).

---

## Build do workspace (colcon)

### `colcon build` do `px4_msgs` falha com `canonicalize_version()`

**Erro:**
```
TypeError: canonicalize_version() got an unexpected keyword argument 'strip_trailing_zero'
Failed <<< px4_msgs
```

**Causa:** versão do `setuptools` incompatível com a geração das mensagens.

**Solução:**
```bash
pip install setuptools==70.3.0
```

### `ModuleNotFoundError: No module named 'cv_bridge'`

**Causa:** falta o pacote de ponte OpenCV ↔ ROS 2.

**Solução:**
```bash
sudo apt install ros-humble-cv-bridge
```

### `_ARRAY_API not found` / `numpy.core.multiarray failed to import`

**Erro:** módulos compilados com NumPy 1.x quebram com NumPy 2.x instalado.

**Solução:** fixar o NumPy na linha 1.x:
```bash
pip install "numpy<2"
```

---

## Comunicação PX4 ↔ ROS 2 / Gazebo

### O drone não se mexe / "comunicação inválida" / sem tópicos do Gazebo

**Sintoma:** nenhum nó do Gazebo aparece em `ros2 topic list`; o drone não responde.

**Solução:** instalar/reinstalar a ponte ros-gz e os pacotes base do ROS:
```bash
sudo apt install ros-humble-ros-gzgarden -y
# se persistir, reinstale o desktop + ferramentas
sudo apt install --reinstall -y ros-humble-desktop ros-dev-tools ros-humble-ros-gzgarden
```

### Conflito da ponte ros-gz (`gzgarden` × `gz`)

**Erro:**
```
ros-humble-ros-gzgarden-bridge : Conflicts: ros-humble-ros-gz-bridge ...
```

**Solução:** remover a variante `gzgarden` conflitante e instalar a `gz`:
```bash
sudo apt remove ros-humble-ros-gzgarden-bridge ros-humble-ros-gzgarden-interfaces
sudo apt install ros-humble-ros-gz-bridge ros-humble-ros-gz-interfaces
```

### `Preflight Fail: ekf2 missing data`

**Causa:** ponte ros-gz ausente/quebrada, faltando dados de sensores no EKF2.

**Solução:** garantir a ponte instalada e o `locale` configurado:
```bash
sudo apt install ros-humble-ros-gzgarden
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

---

## Micro-XRCE-DDS Agent

### `RTPS_READER_HISTORY Error` / erro de payload / conflito de porta

**Erro:**
```
Change payload size of '204' bytes is larger than the history payload size ...
```

**Causa provável:** outro processo já está usando a porta **8888** — o
`MicroXRCEAgent udp4 -p 8888` pode estar subindo em cima da porta que o Gazebo usa.

**Solução:** verifique quem está na porta antes de subir o agente:
```bash
sudo lsof -i:8888
```

---

## Reinstalação limpa do ROS 2 (último recurso)

Quando o ambiente ROS fica inconsistente (fontes/keyrings duplicados), às vezes o mais
rápido é purgar e reinstalar. **Cuidado:** isto apaga o ROS e workspaces.

```bash
# remover pacotes e fontes antigas
sudo apt remove --purge 'ros-*' -y
sudo apt autoremove -y
sudo rm -rf /opt/ros/ ~/.ros
sudo rm -f /etc/apt/sources.list.d/ros2.list /etc/apt/sources.list.d/ros-latest.list
sudo rm -f /usr/share/keyrings/ros2-latest-archive-keyring.gpg /usr/share/keyrings/ros-archive-keyring.gpg
sudo apt clean && sudo rm -rf /var/lib/apt/lists/*

# readicionar a fonte oficial do ROS 2
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" \
  | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update && sudo apt upgrade -y
```

Para conferir fontes/keyrings antigos antes de limpar:
```bash
grep -r "packages.ros.org/ros2/ubuntu" /etc/apt/*
ls /usr/share/keyrings | grep ros
```
