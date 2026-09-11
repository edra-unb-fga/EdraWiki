# Dataset sintético e treinamento (YOLO)

Como a equipe gera datasets e treina os modelos de detecção usados nas missões
(detecção de bases/plataformas e formas coloridas).

!!! info "Fontes"
    [dataset_generator_m1](https://github.com/edra-unb-fga/dataset_generator_m1) ·
    material aprofundado em
    [Producao-do-relatorio-da-SAE](https://github.com/edra-unb-fga/Producao-do-relatorio-da-SAE)
    (Visão Computacional).

---

## Por que dados sintéticos

Coletar e rotular imagens reais é caro e lento. Gerar imagens **sintéticas** resolve isso:

- **Anotações perfeitas** — segmentação, bounding boxes, profundidade e pose saem
  automaticamente, sem erro de rotulagem manual.
- **Diversidade controlada** — variação sistemática de iluminação, materiais, posições.
- **Escalabilidade** — milhares de imagens sem aquisição física.
- **Domain randomization** — variar bastante o sintético melhora a transferência do
  modelo para o mundo real.

---

## Gerador de dataset da equipe (`dataset_generator_m1`)

Gerador **sintético auditável** no formato YOLO para as famílias `landing` e `manometro`
(planos de cena determinísticos, anotações derivadas de alpha-evidence, recipes de
background versionadas, execução resumível).

Instalação e uso (via `uv`):

```bash
uv sync --extra dev
```

Fluxo interativo ponta a ponta:

```bash
# jornada guiada (descobre composers, edita, prepara, roda e exporta)
uv run python -m dataset_generator_m1 start

# validar um composer / catálogo / contrato resolvido
uv run python -m dataset_generator_m1 validate --config examples/configs/landing_minimal.yaml

# gerar (ou retomar) um pool de imagens
uv run python -m dataset_generator_m1 generate \
  --config examples/configs/landing_minimal.yaml \
  --num-images 50 --output-dir outputs/landing-expA --workers auto

# exportar no formato YOLO com splits
uv run python -m dataset_generator_m1 export \
  --pool outputs/landing-expA --format yolo --strategy asset-disjoint \
  --splits train=0.8,val=0.1,test=0.1 --output-dir outputs/landing-yolo
```

!!! tip
    Consulte o `README` e a pasta `docs/` do repositório para as opções completas
    (preflight, preview de backgrounds, benchmark, controle de run ao vivo).

---

## Alternativa: BlenderProc

O [BlenderProc](https://github.com/DLR-RM/BlenderProc) gera dados sintéticos
proceduralmente em cima do Blender. Ideal quando se quer cenas 3D realistas com câmera
simulando a visão do drone (apontada para baixo). Esqueleto do pipeline:

```python
import blenderproc as bproc

bproc.init()
ground = bproc.object.create_primitive("PLANE", size=10)

# câmera simulando a visão do drone (5 m acima, apontada para baixo)
cam = bproc.camera.create_camera_object("MainCamera")
cam.set_location([0, 0, 5])
cam.set_rotation_euler([0, 0, 0])

# iluminação (sol + preenchimento para suavizar sombras)
light = bproc.types.Light(); light.set_type("SUN")
light.set_location([0, 0, 10]); light.set_energy(1.5)
```

---

## Treinamento com YOLO

YOLO (*You Only Look Once*) examina a imagem inteira em uma passagem, prevendo bounding
boxes e classes simultaneamente — bom equilíbrio velocidade/precisão para sistemas
embarcados.

### Estrutura do dataset (formato YOLO)

```
dataset/
├── train/
│   ├── images/   (img_001.jpg, ...)
│   └── labels/   (img_001.txt, ...)
├── val/
│   ├── images/
│   └── labels/
└── test/
    ├── images/
    └── labels/
```

### Arquivo de configuração (`data.yaml`)

```yaml
path: ./dataset
train: train/images
val: val/images
test: test/images

nc: 16   # número de classes
names:
  - circulo_vermelho
  - circulo_verde
  - circulo_azul
  # ... (formas × cores da missão)
```

### Escolha da variante

O modelo tem variantes (**nano, small, medium, large**) que trocam velocidade por
precisão. Para rodar embarcado (Raspberry Pi), as variantes menores (nano/small) são as
mais indicadas.

!!! note "Da simulação para o real"
    Um modelo treinado só com um tipo de base pode **não detectar** as bases de outra
    arena — foi o que travou a detecção na Missão 1 da CBR2025 (o YOLO rodava, mas nunca
    achava as bases da arena nova). Ao trocar de arena/missão, confira se o modelo foi
    treinado com aquelas formas/bases. Veja
    [Simulação CBR2025](../simulacao/cbr2025-arena-missao1.md).

---

## Repositórios relacionados

- [codigo-treinamento-modelo-parametrizado](https://github.com/edra-unb-fga/codigo-treinamento-modelo-parametrizado) — infra de treino parametrizável (Local/Colab/Kaggle).
- [base_detector](https://github.com/edra-unb-fga/base_detector) — modelo treinado para detecção de base da CBR.
- [conversao_e_inferencia_NCNN](https://github.com/edra-unb-fga/conversao_e_inferencia_NCNN) — conversão e inferência otimizada (embarcado).
- [pipeline-criacao-dataset](https://github.com/edra-unb-fga/pipeline-criacao-dataset) — pipeline de criação de dataset.
