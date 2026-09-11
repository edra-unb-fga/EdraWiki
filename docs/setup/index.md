# Setup e Instalação

Guias para preparar o ambiente de desenvolvimento da equipe (PX4, ROS 2, Gazebo,
Docker, ponte uXRCE-DDS).

## Guias disponíveis

<div class="grid cards" markdown>

- :material-penguin:{ .lg } **[Instalação nativa (Linux)](instalacao-nativa.md)**

    ---

    Passo a passo completo no Ubuntu, sem Docker: ROS 2 Humble, PX4 v1.16.0, agente
    Micro-XRCE-DDS, workspace `px4_msgs`, QGroundControl e a arena CBR2025 + fluxo de
    execução e referência de comandos.

- :fontawesome-brands-windows:{ .lg } **[Docker no Windows (WSL2)](docker-windows.md)**

    ---

    Ambiente completo no Windows via WSL2 + Docker Engine + GPU NVIDIA, com todas as
    armadilhas mapeadas (CRLF, GPU, QGC, submódulos).

</div>

## Começando do zero (onboarding)

Se você ainda vai instalar o Ubuntu e configurar Git/SSH pela primeira vez, o tutorial
mais didático é o
[**Configuracoes_Basicas_para_Controle**](https://github.com/edra-unb-fga/Configuracoes_Basicas_para_Controle):

- [`INSTALAÇÃO.md`](https://github.com/edra-unb-fga/Configuracoes_Basicas_para_Controle/blob/main/INSTALA%C3%87%C3%83O.md) — do sistema operacional às dependências.
- [`MISSÃO_TESTE.md`](https://github.com/edra-unb-fga/Configuracoes_Basicas_para_Controle/blob/main/MISS%C3%83O_TESTE.md) — rodar uma missão-teste para validar a instalação.

## Outros repositórios de setup

- [simulation-setup](https://github.com/edra-unb-fga/simulation-setup) — instalação das ferramentas de simulação.
- [uXRCE-configuration](https://github.com/edra-unb-fga/uXRCE-configuration) — ponte PX4 ↔ ROS 2.
- [docker-configuration](https://github.com/edra-unb-fga/docker-configuration) / [ros_humble_docker](https://github.com/edra-unb-fga/ros_humble_docker) — ambientes em container.

> Veja o [Mapa da Documentação](../mapa-da-documentacao.md) para a lista completa.
