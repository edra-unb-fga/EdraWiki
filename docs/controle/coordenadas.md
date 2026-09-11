# Sistemas de coordenadas (PX4 × ROS 2)

Entender os frames de coordenadas é essencial para integrar PX4 e ROS 2 — o **PX4 usa
NED** e o **ROS 2 usa ENU**. Trocar um eixo por engano faz o drone ir para o lado errado.

!!! info "Fonte"
    [Producao-do-relatorio-da-SAE](https://github.com/edra-unb-fga/Producao-do-relatorio-da-SAE)
    · referência: [PX4 — Reference Frames](https://docs.px4.io/main/en/ros2/user_guide.html#ros-2-px4-frame-conventions).

---

## Os frames principais

| Frame | Usado por | Eixo X | Eixo Y | Eixo Z |
|-------|-----------|--------|--------|--------|
| **NED** (North-East-Down) | **PX4** — setpoints de posição/velocidade | Norte | Leste | **Baixo** |
| **ENU** (East-North-Up) | **ROS 2**, RViz | Leste | Norte | **Cima** |
| **FRD** (Front-Right-Down) | body frame do PX4 | Frente | Direita | Baixo |
| **FLU** (Front-Left-Up) | body frame do ROS 2 | Frente | Esquerda | Cima |

!!! warning "Altitude no NED é negativa"
    No NED, o eixo Z aponta para **baixo** — então **maior altitude = valor mais
    negativo**. Um setpoint de "3 metros acima do solo" é `z = -3.0`. Esse é um dos
    tropeços mais comuns ao escrever a lógica de voo.

- Origem: NED e ENU são frames **locais**, com origem geralmente no ponto de decolagem
  (*home position*).

---

## Conversão NED ↔ ENU

A conversão de **posição** troca X↔Y e inverte Z:

```python
def enu_to_ned_position(x_enu, y_enu, z_enu):
    """ENU (ROS 2) -> NED (PX4)."""
    x_ned = y_enu     # Norte = ENU.y
    y_ned = x_enu     # Leste = ENU.x
    z_ned = -z_enu    # Baixo = -ENU.z
    return (x_ned, y_ned, z_ned)

def ned_to_enu_position(x_ned, y_ned, z_ned):
    """NED (PX4) -> ENU (ROS 2)."""
    x_enu = y_ned
    y_enu = x_ned
    z_enu = -z_ned
    return (x_enu, y_enu, z_enu)
```

!!! tip
    A mesma troca (X↔Y, Z invertido) vale para **velocidades**. Como você já envia
    setpoints diretamente em NED para o PX4 (via `trajectory_setpoint`), o cuidado maior é
    lembrar que qualquer coordenada vinda do lado ROS 2 (ex.: RViz, mapas) está em ENU e
    precisa ser convertida antes de virar comando.
