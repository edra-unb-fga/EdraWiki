# Árvores de comportamento (py_trees)

As missões da equipe são estruturadas como **árvores de comportamento** (behavior trees)
usando a biblioteca [`py_trees`](https://py-trees.readthedocs.io/). Elas organizam a
lógica de decisão do drone de forma modular, reativa e legível — mais escalável que uma
máquina de estados para missões complexas.

!!! info "Fonte"
    [Producao-do-relatorio-da-SAE](https://github.com/edra-unb-fga/Producao-do-relatorio-da-SAE)
    (Árvores de comportamento). Exemplos reais em
    [CBR_ws](https://github.com/edra-unb-fga/CBR_ws) e
    [Workspace_Template](https://github.com/edra-unb-fga/Workspace_Template).

---

## Conceitos

Cada nó, ao ser executado (*tick*), retorna um de três estados: **`SUCCESS`**,
**`FAILURE`** ou **`RUNNING`**.

### Tipos de nó

- **Behaviours (folhas)** — a unidade de trabalho: verificar bateria, armar, decolar,
  navegar para um waypoint, detectar objeto.
- **Composites (nós de controle):**
    - **Sequence (`→`)** — executa os filhos em ordem até um **falhar** (E lógico).
    - **Selector (`?`)** — executa os filhos em ordem até um ter **sucesso** (OU lógico /
      fallback).
    - **Parallel** — executa todos os filhos ao mesmo tempo.
- **Decorators** — modificam outro nó: inverter o resultado, tentar de novo após falha,
  aplicar timeout.

### Blackboard

O **blackboard** é um quadro compartilhado onde os nós leem e escrevem dados (posição
atual, alvo detectado, flags de estado) sem acoplar um nó ao outro.

---

## Esqueleto de um behaviour

```python
import py_trees

class CheckBattery(py_trees.behaviour.Behaviour):
    def __init__(self, name="CheckBattery", threshold=20.0):
        super().__init__(name)
        self.threshold = threshold
        self.blackboard = py_trees.blackboard.Blackboard()

    def setup(self, **kwargs):
        self.node = kwargs['node']       # recebe o nó ROS 2
        return True

    def initialise(self):
        pass                              # roda ao (re)entrar no nó

    def update(self):
        if self.blackboard.battery > self.threshold:
            return py_trees.common.Status.SUCCESS
        return py_trees.common.Status.FAILURE
```

## Montando a árvore da missão

```python
def create_mission_tree(node):
    root = py_trees.composites.Selector("MissionControl")

    mission = py_trees.composites.Sequence("MissionSequence")
    preflight = py_trees.composites.Sequence("PreflightChecks")
    # preflight.add_children([CheckBattery(), CheckGPS(), ...])
    # mission.add_children([preflight, ArmDrone(), TakeOff(), NavigateToWaypoint(), ...])

    root.add_children([mission])          # + fallback/failsafe como outro filho do Selector
    return root
```

Com ROS 2, a extensão **`py_trees_ros`** adiciona comportamentos prontos para tópicos,
serviços e ações, além de visualização da árvore em execução.

---

!!! danger "Cuidado: confirmar o estado real, não só o comando enviado"
    Um erro clássico (que travou a Missão 1 da CBR2025) é um nó marcar `SUCCESS` só porque
    **enviou** o comando — sem verificar se o PX4 realmente **aceitou**. Ex.: o nó `Arm`
    dava sucesso mesmo com o armamento negado, e a árvore seguia com o drone parado no
    chão. Sempre que possível, confirme o estado real via `/fmu/out/vehicle_status`
    (armado? em offboard?) antes de retornar `SUCCESS`. Veja o caso completo em
    [Simulação CBR2025](../simulacao/cbr2025-arena-missao1.md).
