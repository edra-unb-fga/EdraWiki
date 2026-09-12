# PX4 + ROS 2 + Gazebo no Windows (Docker/WSL2)

Instalação completa a partir do zero, via **WSL2**. O `docker-compose.yaml` do projeto
[docker-configuration](https://github.com/edra-unb-fga/docker-configuration) foi escrito
para máquinas Linux; este guia mostra como reproduzir fielmente esse ambiente no
Windows, com as armadilhas já mapeadas.

!!! info "Ambiente de referência"
    Validado em **Windows 11** · GPU **NVIDIA RTX 4050** · distro **Ubuntu 22.04** ·
    tempo total **~2h30** · disco **~30 GB**.

!!! abstract "Onde este guia se encaixa"
    Este é um dos **dois** caminhos possíveis para montar o ambiente (veja
    [Primeiros passos](../primeiros-passos.md)). Use este guia **se seu sistema é
    Windows**. Se seu sistema é Linux, use
    [Instalação nativa (Linux)](instalacao-nativa.md) em vez deste — os dois fazem a
    mesma coisa por caminhos diferentes, não é preciso seguir os dois.

**Percurso:** WSL2 → Docker Engine → GPU NVIDIA → projeto e build → QGroundControl →
workspace da missão → rodar → armadilhas.

---

## Por que não usar o Docker Desktop

O `docker-compose.yaml` do projeto monta `/etc/passwd`, `/etc/group`, `/etc/shadow`,
`/home` e `/tmp/.X11-unix` do host, e usa `network_mode: host`. Isso pressupõe um host
Linux real.

No Docker Desktop para Windows os containers rodam dentro da VM `docker-desktop`, não na
sua máquina — o Docker criaria diretórios vazios nesses caminhos e quebraria a resolução
de usuários, e o `network_mode: host` se ligaria à rede da VM, não à sua.

A solução é instalar o **Docker Engine nativo dentro de uma distro Ubuntu no WSL2**.
Assim todos os caminhos, a rede e o X11 se comportam como num Ubuntu de verdade.

!!! tip "Bônus do WSLg"
    O WSLg já fornece `DISPLAY=:0` e o socket `/tmp/.X11-unix/X0`. Você **não** precisa
    de VcXsrv, e o `xhost local:docker` que aparece no README é desnecessário aqui.

---

## Fase 1 — Ubuntu 22.04 no WSL2 (~10 min)

No **PowerShell** do Windows:

```powershell
wsl --install Ubuntu-22.04
```

Na primeira abertura ele pede um nome de usuário e senha do Linux (use de preferência o
mesmo nome do seu usuário do Windows, para reduzir confusão):

```powershell
wsl -d Ubuntu-22.04
```

Confirme que o `systemd` está ativo — o Docker depende dele:

```bash
cat /etc/wsl.conf
```

Deve conter `[boot]` com `systemd=true`. Se não contiver, adicione:

```bash
sudo tee /etc/wsl.conf > /dev/null <<'EOF'
[boot]
systemd=true
EOF
```

E reinicie a distro pelo Windows com `wsl --shutdown`.

!!! note
    A versão 22.04 não é arbitrária: casa com a base do Dockerfile e evita
    incompatibilidades de `glibc` no QGroundControl (Fase 5).

---

## Fase 2 — Docker Engine dentro da distro (~5 min)

Tudo abaixo roda **dentro** do Ubuntu do WSL:

```bash
# repositório oficial da Docker
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release git
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu jammy stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# engine + plugin do compose
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# rodar docker sem sudo
sudo usermod -aG docker $USER

# subir o serviço
sudo systemctl enable --now docker
```

A mudança de grupo só vale em sessões novas. Feche o terminal e abra outro, então confirme:

```bash
docker version --format 'cliente={{.Client.Version}} servidor={{.Server.Version}}'
```

!!! note "Docker Desktop pode continuar instalado"
    Ele não atrapalha, desde que você **não** ative a integração WSL para esta distro.

---

## Fase 3 — GPU NVIDIA (~5 min)

Pule esta fase se sua máquina não tem GPU NVIDIA dedicada — mas note que sem ela o Gazebo
roda em renderização por software e fica a poucos quadros por segundo.

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
  | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
  | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list > /dev/null

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

Valide **antes** de gastar 2 horas de build:

```bash
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```

A tabela do `nvidia-smi` com o nome da sua GPU deve aparecer.

!!! warning
    O driver NVIDIA fica no **Windows**. Não instale driver NVIDIA dentro do WSL: isso
    quebra a passagem de GPU.

---

## Fase 4 — Projeto e build da imagem (~2h)

### 4.1 Clonar dentro do WSL

!!! danger "Armadilha · CRLF"
    Clone **dentro do sistema de arquivos do WSL**, nunca em `/mnt/c/...`. Dois motivos:
    build em `/mnt/c` é drasticamente mais lento, e se o repositório vier com quebras de
    linha CRLF (Git for Windows com `core.autocrlf=true`), o entrypoint vira `#!/bin/bash\r`
    e o container morre com *bad interpreter* — sintoma que só aparece **depois** das 2h de build.

```bash
cd ~
git clone https://github.com/edra-unb-fga/docker-configuration.git
cd docker-configuration
```

Se você copiou de uma pasta do Windows em vez de clonar, normalize antes:

```bash
sed -i 's/\r$//' ros_entrypoint.sh start.sh humble-px4.Dockerfile
chmod +x ros_entrypoint.sh start.sh
git config core.autocrlf false
```

### 4.2 Correções necessárias no repositório

Verifique se já foram corrigidos antes de aplicar:

| Arquivo | Problema | Correção |
|---------|----------|----------|
| `humble-px4.Dockerfile` | `COPY ../ros_entrypoint.sh /` sai do build context | `COPY ros_entrypoint.sh /` |
| `.env.example` | define `HOST_USERNAME`, mas o compose lê `${HOST_USER}` | usar `HOST_USER=...` |
| `humble-px4.Dockerfile` | `docker exec` não passa pelo ENTRYPOINT, então o ROS 2 não fica no PATH | acrescentar a linha abaixo |

No fim do `Dockerfile`, logo após o `chmod` do entrypoint:

```dockerfile
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> /root/.bashrc
```

### 4.3 Configuração local

```bash
echo "HOST_USER=$USER" > .env
mkdir -p ~/Volumes
```

### 4.4 Override específico do WSL

Este arquivo não existe no repositório: crie-o. Ele garante OpenGL acelerado por GPU:

```bash
cat > docker-compose.wsl.yaml <<'EOF'
# Override para WSL2 — não usar em hosts Linux nativos.
services:
  ros-px4-humble:
    restart: unless-stopped
    volumes:
      - /usr/lib/wsl:/usr/lib/wsl:ro
    environment:
      - LD_LIBRARY_PATH=/usr/lib/wsl/lib
      - MESA_D3D12_DEFAULT_ADAPTER_NAME=NVIDIA
EOF
```

!!! note "Por que isso importa"
    O container recebe apenas `libdxcore.so` do toolkit da NVIDIA. Montando `/usr/lib/wsl`
    ele passa a enxergar `libd3d12.so`, e o Mesa usa o driver Gallium `d3d12` para
    renderizar na GPU. Sem isso, o Gazebo cai em `llvmpipe` (software).

### 4.5 Build

```bash
docker compose \
  -f docker-compose.yaml \
  -f docker-compose.nvidia.yaml \
  -f docker-compose.wsl.yaml \
  up -d --build
```

O build leva de **1h30 a 2h** e a imagem final ocupa cerca de **25 GB**. Confirme o espaço
antes de começar.

Verificar o resultado:

```bash
docker exec ros-px4-humble nvidia-smi --query-gpu=name --format=csv,noheader
docker exec ros-px4-humble bash -c 'apt-get install -y -qq mesa-utils && glxinfo -B | grep -E "renderer|Accelerated"'
```

O resultado correto é `OpenGL renderer string: D3D12 (NVIDIA ...)` e `Accelerated: yes`.

!!! danger "Armadilha · start.sh"
    Não use o `./start.sh` do repositório. Ele só usa dois dos três arquivos de compose,
    então você perde a aceleração gráfica sem perceber.

---

## Fase 5 — QGroundControl (~15 min)

O QGC é **obrigatório**: sem uma estação de controle conectada, o PX4 recusa o armamento
com `Preflight Fail: No connection to the GCS` e o drone nunca decola.

!!! danger "Armadilha · instale no WSL, não no Windows"
    O PX4 loga `MAVLink only on localhost`, ou seja, transmite para `127.0.0.1:14550`.
    Como o container usa `network_mode: host`, esse loopback é o do Ubuntu do WSL. Um QGC
    rodando no Windows simplesmente não recebe esses pacotes.

### 5.1 Use a linha v4.4.x, não a v5

As builds v5 exigem `GLIBC 2.38` e o Ubuntu 22.04 tem 2.35. A v4.4.5 é compilada para 22.04:

```bash
cd ~
curl -fL -o QGroundControl.AppImage \
  https://github.com/mavlink/qgroundcontrol/releases/download/v4.4.5/QGroundControl.AppImage
chmod +x QGroundControl.AppImage
```

### 5.2 Dependências

```bash
sudo apt-get install -y \
  libfuse2 libsdl2-2.0-0 libpulse0 libpulse-mainloop-glib0 \
  libgstreamer1.0-0 libgstreamer-gl1.0-0 gstreamer1.0-gl \
  gstreamer1.0-plugins-base gstreamer1.0-plugins-good \
  gstreamer1.0-plugins-bad gstreamer1.0-libav \
  libxcb-shape0 libxcb-cursor0 libxcb-icccm4 libxcb-image0 \
  libxcb-keysyms1 libxcb-render-util0 libxcb-xinerama0 libxkbcommon-x11-0
sudo usermod -aG dialout $USER
```

!!! note "O detalhe que trava todo mundo"
    O `libxcb-shape0` é o mais importante da lista. Sem ele o plugin `xcb` fica invisível e
    o QGC aborta dizendo que *nenhum plugin de plataforma pôde ser inicializado*, mesmo
    estando presente no bundle.

### 5.3 Launcher

Crie um atalho para forçar o plugin `xcb`:

```bash
cat > ~/qgc.sh <<'EOF'
#!/bin/bash
export DISPLAY=:0
export XDG_RUNTIME_DIR=/run/user/$(id -u)
export QT_QPA_PLATFORM=xcb
exec "$HOME/QGroundControl.AppImage" "$@"
EOF
chmod +x ~/qgc.sh
```

A partir do Windows, abra com uma linha: `wsl -d Ubuntu-22.04 -- ~/qgc.sh`

---

## Fase 6 — Workspace da missão (~10 min)

O [Workspace_Template](https://github.com/edra-unb-fga/Workspace_Template) é privado.
Configure o git do WSL para reaproveitar o Git Credential Manager do Windows (assim o
clone por HTTPS autentica sem chave SSH):

```bash
git config --global credential.helper \
  '/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe'
git config --global credential.'https://github.com'.provider github
```

Clone no home do WSL, **não** dentro do container:

```bash
cd ~
git clone https://github.com/edra-unb-fga/Workspace_Template.git
cd Workspace_Template
git submodule update --init --recursive
```

!!! warning "Armadilha · submódulo"
    O `src/px4_msgs` é um submódulo e o README não menciona isso. Sem o
    `git submodule update --init --recursive`, o `colcon build` falha por falta das
    mensagens do PX4.

Como o compose monta `/home` inteiro, o workspace aparece dentro do container no mesmo
caminho. Compile **de dentro do container**:

```bash
docker exec -it ros-px4-humble bash
cd /home/SEU_USUARIO/Workspace_Template
colcon build
```

!!! warning "Armadilha · compile com a simulação desligada"
    O `colcon build` do `px4_msgs` satura todos os núcleos. Se o Gazebo estiver no ar, o
    PX4 perde o sincronismo e derruba a simulação com `ERROR [vehicle_imu] timestamp error`.
    Compile primeiro, simule depois.

---

## Rodar a simulação

O que se repete e o que não:

| Comando | Frequência |
|---------|-----------|
| `docker compose ... up -d` | **Uma vez só.** O container fica no ar até você derrubá-lo. |
| `docker exec -it ros-px4-humble bash` | **Em cada terminal** que precise de um shell no container. |

Atalho de uma linha, direto do Windows: `wsl -d Ubuntu-22.04 -- docker exec -it ros-px4-humble bash`

!!! tip
    Você sabe que está *dentro* do container porque o prompt vira `root@...:/edra#`. Se
    ainda mostrar `/mnt/c/...`, você está no Ubuntu do WSL.

### Ordem de abertura

Abra o QGroundControl **antes** de rodar a missão.

=== "Terminal 1 — PX4 + Gazebo"

    ```bash
    cd /edra/PX4-Autopilot
    make px4_sitl gz_x500_mono_cam_down
    ```
    !!! danger "Armadilha · caminho do PX4"
        O README diz `cd ~/PX4-Autopilot`, que **não existe**. O WORKDIR da imagem é
        `/edra`, e dentro do container você é `root` — então `~` aponta para `/root`. Use
        sempre o caminho absoluto `/edra/PX4-Autopilot`.

    Este terminal precisa ser interativo: é nele que aparece o console `pxh>`.

=== "Terminal 2 — ponte MicroXRCE"

    ```bash
    MicroXRCEAgent udp4 -p 8888
    ```

=== "Terminal 3 — QGroundControl (no Windows)"

    ```bash
    wsl -d Ubuntu-22.04 -- ~/qgc.sh
    ```

=== "Terminal 4 — missão ROS 2"

    ```bash
    cd /home/SEU_USUARIO/Workspace_Template
    source install/setup.bash
    ros2 run missao_template main
    ```

### Conferir que está tudo vivo

```bash
# de dentro do container
ros2 topic list | grep /fmu/out | wc -l     # esperado: dezenas
ros2 topic echo /fmu/out/vehicle_status_v4 --qos-reliability best_effort
```

!!! note "A pegadinha mais comum do PX4 com ROS 2"
    O PX4 publica em **best-effort** e o `ros2 topic echo` assume **reliable** por padrão.
    Sem a flag `--qos-reliability best_effort`, o comando fica mudo mesmo com a telemetria
    perfeita, e você perde tempo caçando um problema que não existe.

---

## Armadilhas — sintomas e causas

| Sintoma | Causa e correção |
|---------|------------------|
| `container is not running` | O daemon do WSL pode ser parado pelo ciclo de energia do Windows. Com `restart: unless-stopped` ele volta sozinho; senão, refaça o `up -d`. |
| `ros2: command not found` | Shell não-interativo não lê o `.bashrc`. Em scripts, use `source /opt/ros/humble/setup.bash`. |
| `bad interpreter` / `exec format error` | Quebras CRLF nos `.sh`. Rode `sed -i 's/\r$//'` nos scripts e no Dockerfile. |
| Gazebo a poucos FPS | Faltou o `docker-compose.wsl.yaml`. Confirme com `glxinfo -B`: se aparecer `llvmpipe`, é software. |
| `Arming denied` / drone parado | `Preflight Fail: No connection to the GCS`. Abra o QGroundControl. Alternativa só para simulação: `param set NAV_DLL_ACT 0` no console `pxh>`. |
| `ERROR [vehicle_imu] timestamp error` | CPU saturada — quase sempre um `colcon build` rodando junto. Não compile com a simulação no ar. |
| QGC não abre janela | Falta `QT_QPA_PLATFORM=xcb` ou a lib `libxcb-shape0`. |
| `ros2 topic echo` mudo | Falta `--qos-reliability best_effort`, ou o `px4_msgs` não está compilado. |
| Log do PX4 crescendo em GB | PX4 rodando destacado, sem TTY. Rode o Terminal 1 sempre interativo. |

!!! tip "Sobre a árvore de comportamento da missão"
    O template marca `Arm ✓` mesmo quando o PX4 **recusa** o armamento, porque o nó
    confirma apenas que enviou o comando, não que ele foi aceito. Se o drone não sai do
    chão mas a árvore mostra tudo verde, verifique o Terminal 1: a recusa aparece lá, não
    na missão.

---

*Procedimento reconstruído a partir de uma instalação real completa, em Windows 11 com
RTX 4050. Os tempos e tamanhos são medidos, não estimados. Ajuste `SEU_USUARIO` para o
nome do seu usuário do Linux.*
