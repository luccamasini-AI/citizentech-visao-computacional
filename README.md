# CitizenTech — Módulo de Visão Computacional para Manutenção Urbana

Sistema de Visão Computacional baseado em Transferência de Aprendizado (`MobileNetV3-Small`) e Explicabilidade Espacial (`Grad-CAM`) para identificação preventiva de fissuras em superfícies de concreto e simulação de monitoramento urbano contínuo em vídeo.

---

## Integrantes do Grupo

1. **Lucca Phelipe Masini** — RM 564121
2. **Igor Paixão Sarak** — RM 563726
3. **Bernardo Braga Perobeli** — RM 562468

---

## 1. Contextualização e Escopo Técnico

A identificação precoce de fissuras visíveis em estruturas de concreto (pontes, viadutos, muros de contenção e calçadas) auxilia a administração pública a priorizar vistorias de engenharia e planejar ações de manutenção. Este projeto integra ao protótipo **CitizenTech** um módulo completo de triagem visual automatizada com duas frentes operacionais:

- **Triagem de Imagem Individual:** Classifica superfícies de concreto entre *Com Fissura (Positive)* e *Sem Fissura (Negative)*, apresentando a probabilidade da predição, o grau de confiança do modelo, o mapa de atenção espacial via *Grad-CAM* e a caixa delimitadora (*bounding box*) sobre a região crítica identificada.
- **Monitoramento Urbano Contínuo (Vídeo e Sequência de Quadros):** Processa vídeos curtos (`.mp4`, `.avi`, `.mov`) ou sequências ordenadas de imagens de inspeção viária, registrando telemetria em tempo real (latência por quadro em milissegundos e taxa de quadros por segundo — FPS), diagnóstico consolidado do trecho, inspetor individual de quadros e tabela auditável de ocorrências.

> **Diretriz de Governança e Engenharia Civil:** Esta solução atua exclusivamente como ferramenta de **triagem visual preventiva** para priorização de vistorias presenciais. A presença de fissuras superficiais em imagens bidimensionais, isoladamente, não permite determinar a profundidade do dano, sua origem patológica ou a segurança estrutural da edificação.

---

## 2. Estrutura do Repositório

```text
.
├── app_citizentech.py                      # Aplicação interativa do protótipo CitizenTech (Streamlit)
├── GS_FRONT.ipynb                          # Notebook executado (Preparação, Treinamento, Avaliação e Benchmark)
├── relatorio_tecnico_citizentech.html      # Relatório Técnico formatado para impressão A4 / exportação PDF
├── Iniciar_CitizenTech.bat                 # Inicializador automatizado da aplicação para Windows
├── requirements.txt                        # Dependências de execução do ambiente local
├── artifacts/
│   ├── figures/                            # Gráficos gerados na avaliação do conjunto de teste
│   │   ├── confusion_matrix.png            # Matriz de confusão no conjunto de teste (678 imagens)
│   │   ├── training_history.png            # Curvas de perda (Loss) e acurácia no treinamento e validação
│   │   ├── correct_predictions_analysis.png# Exemplos de acertos acompanhados de mapas Grad-CAM
│   │   └── error_and_interference_analysis.png # Análise visual dos falsos negativos e casos limítrofes
│   ├── metrics/
│   │   └── evaluation_report.json          # Métricas consolidadas e benchmarks de hardware (GPU e CPU)
│   ├── model/
│   │   └── concrete_crack_mobilenetv3.pth  # Pesos treinados da arquitetura MobileNetV3-Small
│   └── sample_media/
│       ├── simulacao_monitoramento_urbano.mp4 # Vídeo curto de simulação de vistoria urbana
│       └── test_images/                    # Amostras reais do conjunto de teste para demonstração
└── src/
    ├── domain/
    │   └── inspection_contracts.py         # Contratos de dados imutáveis e enumerações de domínio
    ├── inference/
    │   └── crack_classifier_service.py     # Serviço de inferência vetorial, fusão Conv-BN e Grad-CAM
    ├── reporting/
    │   ├── bundle_and_report_manager.py    # Gerenciador seguro de artefatos e validação contra Zip Slip
    │   └── generate_printable_html_report.py # Gerador do relatório técnico A4 com gráficos em Base64
    └── video/
        └── urban_monitoring_processor.py   # Processador de fluxos de vídeo e renderizador de telemetria HUD
```

---

## 3. Metodologia e Prevenção de Vazamento de Dados

1. **Base de Dados:** *Concrete Crack Images for Classification* (Özgenel, 2019 — Mendeley Data), composta originalmente por 40.000 imagens RGB (`227x227 px`).
2. **Amostragem Balanceada:** Utilizou-se uma amostra de **4.000 imagens** (`2.000 Positive` e `2.000 Negative`), dividida em **Treinamento (2.642)**, **Validação (680)** e **Teste (678)**.
3. **Particionamento por Superfície de Origem (*Zero Data Leakage*):** Para evitar que recortes oriundos de uma mesma fotografia original fossem distribuídos entre conjuntos distintos, aplicou-se `GroupShuffleSplit` agrupado pelo identificador da estrutura de origem, garantindo interseção nula de grupos entre treino, validação e teste.
4. **Pré-processamento e Aumento de Dados:** Redimensionamento para `224x224 px`, normalização pelos parâmetros canônicos da ImageNet (`mean = [0.485, 0.456, 0.406]`, `std = [0.229, 0.224, 0.225]`) e aplicação de *Data Augmentation* no conjunto de treino (espelhamento horizontal/vertical, rotação de ±15° e *ColorJitter*).

---

## 4. Resultados Experimentais e Desempenho Computacional

### 4.1. Métricas de Qualidade no Conjunto de Teste (678 Imagens Inéditas)

| Métrica | Resultado Obtido |
| :--- | :--- |
| **Precisão (Precision)** | `100,00%` (`FP = 0`) |
| **Sensibilidade (Recall)** | `99,41%` (`FN = 2`) |
| **F1-Score** | `99,70%` |
| **Acurácia Global** | `99,71%` (`TN = 339`, `TP = 337`) |
| **ROC-AUC** | `1,0000` |
| **Limiar de Confiança (Threshold)** | `0,50` |

### 4.2. Benchmark de Latência e Taxa de Quadros (FPS)

| Ambiente de Hardware | Resolução | Latência Média / Imagem | Latência P95 | Taxa de Processamento |
| :--- | :--- | :--- | :--- | :--- |
| **GPU NVIDIA Tesla T4 (Google Colab)** | `224x224 px` | `4,48 ms` (±0,52 ms) | `5,35 ms` | `223,2 FPS` |
| **CPU Local (Windows AMD64 — Motor Vetorizado)** | `224x224 px` | `30,72 ms` (±0,74 ms) | `31,65 ms` | `32,6 FPS` |

---

## 5. Como Executar o Protótipo Localmente

### Opção 1 — Inicialização Rápida (Windows)

Execute o script de inicialização na raiz do projeto:

```bat
Iniciar_CitizenTech.bat
```

O script valida a instalação do Python, instala automaticamente as dependências listadas em `requirements.txt` caso necessário e abre a interface web do **CitizenTech** no navegador padrão.

### Opção 2 — Execução Manual via Terminal

```bash
pip install -r requirements.txt
streamlit run app_citizentech.py
```

> **Portabilidade de Execução:** O serviço de inferência (`src/inference/crack_classifier_service.py`) conta com um motor vetorial próprio com fusão matemática das camadas *Conv2d + BatchNorm2d*, carregando diretamente os tensores treinados do arquivo `.pth` sem exigir a instalação de drivers CUDA em máquinas locais de avaliação.

---

## 6. Relatório Técnico Imprimível (A4 / PDF)

O relatório técnico consolidado encontra-se no arquivo `relatorio_tecnico_citizentech.html`. Para exportá-lo em PDF, abra o arquivo em qualquer navegador web e utilize o botão de impressão no topo da página (ou pressione `Ctrl + P` selecionando **Salvar como PDF**).
