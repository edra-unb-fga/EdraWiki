# Especificação e Requisitos de Visão Computacional - Fase 2 / Teseu

### 1. Identificação do Projeto

* **Fase(s) da Competição:**
    * [ ] **Fase 1 - Mapeamento e Detecção de Bases**
    * [x] **Fase 2 - Detecção e Rastreamento de Kit de Primeiros Socorros**
    * [ ] **Fase 3 - Interação Humano-Robô (Reconhecimento de Gestos/Operador)**
    * [ ] **Fase 4 - Leitura de QR Codes em Ambientes Confinados/Escuros**
* **Plataforma e Câmera(s):**
    * [x] **X500 (Fases 1, 2, 3)** - *Câmera(s): Câmera inferior RGB (modelo, resolução e FOV a confirmar)*
    * [ ] **Tello / Mini UAV (Fases 1, 3, 4)**
* **Responsáveis da Visão Computacional:** `Marcelo Makoto Araki Takechi`
* **Data:** 09/10/2026


---

### 2. Entendimento e Desafio Visual da Missão

#### 2.1 Descrição do Desafio de Visão
O drone deve transportar **3 kits de primeiros socorros**, um por vez, de bases no topo do ambiente da Fase 4 (a **1,5 m** de altura) para **3 bases vazias** espalhadas pela arena (qualquer posição, altura de 0 a 1,5 m, posições conhecidas antecipadamente).

A câmera inferior precisa:

1. **Detectar o kit sem nenhum marcador** (QR, código de barras ou AprilTag são proibidos) e estimar posição e orientação. O kit pode estar em **qualquer orientação** dentro do círculo amarelo (Ø 0,8 m) e mede ~19,5 × 8 × 7 cm, prateado com bordas pretas (Fig. 8).
2. **Detectar a base de pouso** (quadrado azul 1 × 1 m, círculo amarelo de 0,8 m, cruz de 0,5 m, linhas de 5 cm) e estimar o centro, pois **todo o trem de pouso deve ficar dentro da base** na coleta e na entrega.
3. **Verificar se a base de destino está vazia**, pois é proibido entregar em base com caixa ou outro kit.
4. **Confirmar coleta e entrega** (kit levantado; kit dentro da base de destino), que definem a pontuação (+40 / +20 / −5).

#### 2.2 Fluxo Operacional de Processamento (Passo a Passo)
1. **Captura do Frame:** câmera inferior (nadir), 640 × 480 a ≥ 30 FPS na captura e processamento a ≥ 15 FPS. Exposição automática com limite de tempo de exposição para reduzir o motion blur causado por vibração.
2. **Pré-processamento:** correção de distorção (intrínsecos calibrados com tabuleiro), conversão para HSV, equalização adaptativa (CLAHE) no canal V e máscara da região ocupada pela garra/trem de pouso.
3. **Detecção / Inferência:**
    * **Kit:** detector com caixa orientada (YOLOv8n-OBB), que entrega centro e ângulo do kit diretamente.
    * **Base:** segmentação HSV do azul e do amarelo, ajuste de elipse no círculo amarelo e centro da cruz (método clássico, sem treino), com o detector como redundância.
    * **Ocupação da base:** presença de kit ou caixa dentro da região do círculo amarelo.
4. **Pós-processamento e Saída:** confirmação temporal (≥ 5 frames consecutivos), filtro de Kalman sobre (x, y, yaw), conversão de pixels para metros usando a altitude (`dx = (u − cx) · h / fx`) e o extrínseco câmera→garra. Publicação do erro relativo (dx, dy, dyaw) e dos estados (kit detectado, preso, entregue; base vazia ou ocupada).

#### 2.3 Restrições e Desafios de Imagem na Arena
* **Marcadores:** **proibido** qualquer marcador no kit (Fase 2 – Condições). Sem cabo suspenso e sem placa magnética.
* **Condições de Iluminação:** luz solar externa variável e não uniforme, piso liso que pode refletir (Arena Obs. 1 e 4). O kit prateado é **especular**, e sua aparência muda muito com a luz. Os parâmetros **não podem ser alterados após a 1ª tentativa** (O Desafio – Obs. 1), então o pipeline precisa se adaptar sozinho.
* **Perspectiva e Oclusão:** altura relativa câmera–alvo de ~3 m até ~0,2 m durante a descida. Com HFOV ~70°, a base inteira (1 m) só cabe no quadro acima de ~0,7 m, e abaixo de ~0,25 m o kit sai do quadro ou desfoca. **O kit preso na garra pode tampar a câmera durante a entrega.** Há ainda pessoas circulando, banner, rede de proteção, propwash e vibração.

---

### 3. Requisitos de Visão Computacional

> **Prioridade:** **P0** (Crítico / Obrigatório) | **P1** (Desejável / Melhoria)

| ID | Descrição do Requisito de Visão | Pri. | Método de Validação |
| :--- | :--- | :--- | :--- |
| **REQ-CV-01** | Detectar o kit sem marcadores, em qualquer orientação: precisão ≥ 95% e recall ≥ 95% com o kit girado de 0° a 345° (passo de 15°) e câmera a 0,4–2,0 m (Interno). Uso de marcadores = 0 (CBR) | **P0** | Conjunto de teste real rotulado (≥ 300 imagens) |
| **REQ-CV-02** | Estimar a pose do kit em relação à garra: erro do centróide ≤ 3 cm e erro de yaw ≤ 10° (módulo 180°) com câmera a 0,5 m (Interno) | **P0** | Bancada com gabarito medido |
| **REQ-CV-03** | Detectar a base e estimar o centro: detecção ≥ 98% entre 0,8 e 3,0 m acima da base; erro de centro ≤ 10 cm a 1,5 m (Interno). Trem de pouso 100% dentro de 1 × 1 m (CBR) | **P0** | Bancada + pousos reais medidos |
| **REQ-CV-04** | Classificar a base de destino como vazia ou ocupada: acurácia ≥ 99%, com 0 casos de base ocupada classificada como vazia em 100 testes (Interno) | **P1** | Dataset de bases vazias/ocupadas |
| **REQ-CV-05** | Confirmar coleta (kit preso e fora da base de origem) em ≤ 2 s após subir 30 cm, com acerto ≥ 98% (Interno) | **P0** | 50 ciclos de pega, incluindo falhas propositais |
| **REQ-CV-06** | Confirmar entrega (kit dentro da base de destino) com acerto ≥ 98% (Interno) | **P1** | 50 soltas, incluindo kit na borda e fora |
| **REQ-CV-07** | Processar em tempo real no computador de bordo: ≥ 15 FPS e latência ≤ 65 ms da câmera até a publicação (Interno) | **P0** | Benchmark de 10 min no hardware do Teseu |
| **REQ-CV-08** | Manter o desempenho sob iluminação não uniforme: queda ≤ 5 p.p. nas métricas de REQ-01/03 em 4 condições de luz, com 0 parâmetros alterados entre tentativas (CBR) | **P0** | Repetição de testes com configuração congelada |
| **REQ-CV-09** | Limitar falsos positivos: ≤ 1 falso positivo de kit por minuto de voo sobre a arena sem kits e com pessoas circulando (Interno) | **P1** | Gravação de 5 min sem kits |
| **REQ-CV-10** | Manter o rastreamento na descida: perda de track ≤ 0,5 s até a altura mínima da câmera (~15–25 cm, a definir); abaixo disso, usar a última estimativa filtrada (Interno) | **P1** | Análise de gravações de descidas |
| **REQ-CV-11** | Registrar rosbag em 100% das tentativas com imagem (≥ 5 Hz, comprimida) e todos os tópicos `/vision/*` com timestamp (CBR exige o log; taxa Interna) | **P0** | Checklist pós-tentativa (`ros2 bag info`) |
| **REQ-CV-12** | Exibir no terminal cada evento da visão (kit identificado, kit preso, entregue, base vazia/ocupada) em ≤ 1 s (Interno) | **P1** | Teste integrado observando o terminal |

---

### 4. Pipeline do Dataset e Modelo de Visão

#### 4.1 Estratégia de Dados e Coleta
* **Classes de Detecção:** `kit_primeiros_socorros` (caixa orientada) e `base_pouso`. A ocupação da base é derivada da presença do kit dentro da base, sem classe própria.
* **Tamanho e Origem do Dataset:**
    * ~1.500 imagens **reais**, gravadas com a própria câmera do Teseu (montada no drone ou em suporte na altura certa), com 2 a 3 kits iguais aos da Fig. 8.
    * ~1.500 imagens **simuladas** (Gazebo) para variar fundo, posição e luz.
    * **Teste:** ≥ 300 imagens reais **separadas do treino**, cobrindo alturas de 0,4 a 3 m, ângulos de 0° a 345° e 4 condições de luz.
    * Incluir imagens “negativas”: bases vazias, pessoas, banner, rede, piso com reflexo.
* **Técnicas de Data Augmentation:** rotação de 0° a 360°, escala (simula altura), variação de brilho/contraste/gama, sombras sintéticas, **brilhos especulares sintéticos** (kit prateado), motion blur, ruído gaussiano, cortes parciais (kit saindo do quadro) e oclusão parcial pela garra.

#### 4.2 Modelo / Algoritmo Escolhido
* **Arquitetura Visual:**
    * **Kit:** YOLOv8n-OBB (caixa orientada fornece o yaw sem pós-processamento extra). Se o computador de bordo não sustentar 15 FPS, usar resolução de 320 px ou ROI em torno da última detecção.
    * **Base:** OpenCV (máscara HSV azul + amarelo, `fitEllipse` no círculo, centro da cruz) como método principal; o detector serve de redundância.
    * **Filtragem:** filtro de Kalman em (x, y, yaw) + confirmação por N = 5 frames.
* **Métricas Esperadas:** mAP@50 ≥ 0,90 no kit; precisão e recall ≥ 95% (REQ-01); inferência ≤ 50 ms por frame; pipeline completo ≤ 65 ms.

---

### 5. Interfaces de Software e Tópicos ROS 2

* **Nó ROS 2:** `vision_kit_detector_node`
* **Tópicos de Entrada (Inscrição):**
    * `/camera/image_raw` `[sensor_msgs/msg/Image]` - Imagem bruta da câmera inferior.
    * `/camera/camera_info` `[sensor_msgs/msg/CameraInfo]` - Intrínsecos para desdistorção e conversão pixel→metro.
    * `/fmu/out/vehicle_local_position` `[px4_msgs/msg/VehicleLocalPosition]` - Altitude, para converter o erro em metros (ou o tópico de sensor de distância, se houver).
    * `/mission/vision_mode` `[std_msgs/msg/String]` - Modo pedido pela missão: `BUSCAR_KIT`, `ALINHAR_BASE`, `VERIFICAR_COLETA`, `VERIFICAR_ENTREGA`.
* **Tópicos de Saída (Publicação):**
    * `/vision/target_pose_error` `[geometry_msgs/msg/Vector3]` - Erro relativo do alvo atual em relação à garra (`x = dx` [m], `y = dy` [m], `z = dyaw` [rad]).
    * `/vision/detection_status` `[std_msgs/msg/Bool]` - Alvo atual confirmado (≥ 5 frames).
    * `/vision/kit_state` `[std_msgs/msg/String]` - `KIT_DETECTADO`, `KIT_PRESO`, `KIT_PERDIDO`, `KIT_ENTREGUE`.
    * `/vision/base_status` `[std_msgs/msg/String]` - `BASE_VAZIA` / `BASE_OCUPADA`.
    * `/vision/debug_image` `[sensor_msgs/msg/Image]` - Imagem anotada (caixas, centro, ângulo), para o rosbag e para o juiz.

> O tópico `/vision/detected_command` do template não se aplica à Fase 2.

---

### 6. Plano de Verificação e Validação (V&V)

| ID Teste | Requisito Relacionado | Procedimento do Teste | Critério de Sucesso | Status |
| :--- | :--- | :--- | :--- | :--- |
| **TST-CV-01** | REQ-CV-01 | Rodar o detector no conjunto de teste real (≥ 300 imagens) com o kit girado de 0° a 345° (passo de 15°) e câmera a 0,4–2,0 m; calcular P/R por faixa. | Precisão e recall ≥ 95% em todas as faixas. | [ ] Não iniciado |
| **TST-CV-02** | REQ-CV-02 | Bancada: câmera fixa a 0,5 m; kit em 20 posições/ângulos medidos com trena e transferidor; comparar com `/vision/target_pose_error`. | Erro do centróide ≤ 3 cm e yaw ≤ 10° em ≥ 95% dos casos. | [ ] Não iniciado |
| **TST-CV-03** | REQ-CV-03 | Base real fotografada de 0,8 a 3,0 m; depois 20 pousos em voo medindo o deslocamento do trem de pouso até o centro. | Detecção ≥ 98%; erro de centro ≤ 10 cm a 1,5 m; 20/20 pousos com o trem 100% dentro da base. | [ ] Não iniciado |
| **TST-CV-04** | REQ-CV-04 | 100 imagens de bases vazias e ocupadas (kit e caixa em posições variadas, 4 condições de luz). | Acurácia ≥ 99%; 0 bases ocupadas classificadas como vazias. | [ ] Não iniciado |
| **TST-CV-05** | REQ-CV-05, REQ-CV-06 | 50 ciclos de pega e 50 de solta (incluindo pegas falhas, kit na borda e fora da base); comparar `/vision/kit_state` com o resultado real. | Acerto ≥ 98%; confirmação de coleta em ≤ 2 s. | [ ] Não iniciado |
| **TST-CV-06** | REQ-CV-07 | Rodar o pipeline completo por 10 min no computador de bordo do Teseu, registrando FPS, latência (timestamps), CPU/GPU e temperatura. | FPS ≥ 15 (média e p5); latência p95 ≤ 65 ms; sem throttling térmico. | [ ] Não iniciado |
| **TST-CV-07** | REQ-CV-08 | Repetir TST-CV-01 e TST-CV-03 com a configuração congelada em 4 condições: luz uniforme, sol lateral, sombra parcial e reflexo no piso/kit (registrar lux). | Queda ≤ 5 p.p. em relação à luz uniforme; nenhum parâmetro alterado. | [ ] Não iniciado |
| **TST-CV-08** | REQ-CV-09 | Voar ou gravar 5 min sobre a arena sem kits, com pessoas circulando e banner no fundo. | ≤ 1 falso positivo confirmado por minuto. | [ ] Não iniciado |
| **TST-CV-09** | REQ-CV-10 | Gravar 20 descidas reais sobre o kit; medir a altura em que a detecção cai e a maior lacuna de track. | Lacunas ≤ 0,5 s acima da altura mínima definida. | [ ] Não iniciado |
| **TST-CV-10** | REQ-CV-11, REQ-CV-12 | Ensaio completo de 10 min com regras da competição (config congelada, 3 tentativas); conferir rosbag e terminal ao final. | Rosbag presente em 3/3 tentativas com todos os tópicos; todos os eventos impressos em ≤ 1 s. | [ ] Não iniciado |
