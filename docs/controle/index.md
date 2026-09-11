# Controle · PX4 · Behavior Trees

Arquitetura de controle, interação com o PX4 e lógica de missão (árvores de
comportamento).

## Guias disponíveis

<div class="grid cards" markdown>

- :material-drone:{ .lg } **[Modo Offboard: posição × velocidade](offboard.md)**

    ---

    Como funciona o controle externo do PX4 via ROS 2, os tópicos essenciais, a
    diferença entre controlar por posição e por velocidade, e o erro clássico das
    flags `OffboardControlMode`.

- :material-axis-arrow:{ .lg } **[Sistemas de coordenadas (NED × ENU)](coordenadas.md)**

    ---

    Os frames do PX4 (NED) e do ROS 2 (ENU), por que a altitude é negativa, e as
    conversões entre eles.

- :material-file-tree:{ .lg } **[Árvores de comportamento (py_trees)](behavior-trees.md)**

    ---

    Como as missões são estruturadas: Sequence, Selector, Parallel, blackboard, e o
    cuidado de confirmar o estado real do drone.

</div>

## Documentação existente (a migrar)

O repositório [Producao-do-relatorio-da-SAE](https://github.com/edra-unb-fga/Producao-do-relatorio-da-SAE)
tem material aprofundado de Controle & Sistemas Embarcados, incluindo **arquitetura PX4**
e **hierarquias de controle / failsafes**.

> Veja o [Mapa da Documentação](../mapa-da-documentacao.md) para a lista completa.
