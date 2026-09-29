# FarmTech Vision | Visão computacional com YOLO e CNN

**FIAP | Inteligência Artificial | Fase 6, Cap 1: O despertar da Rede Neural**

[![Notebook](https://img.shields.io/badge/Jupyter-Notebook%20executado-F37626?logo=jupyter&logoColor=white)](FelipeShimoda_rm573344_pbl_fase6.ipynb)
[![Colab](https://img.shields.io/badge/Google%20Colab-GPU%20T4-F9AB00?logo=googlecolab&logoColor=white)](https://colab.research.google.com/github/dev-felipeshimoda/Fase-6---Cap-1---O-despertar-da-Rede-Neural/blob/main/FelipeShimoda_rm573344_pbl_fase6.ipynb)
[![YOLOv5](https://img.shields.io/badge/YOLOv5-Ultralytics-0B23A9)](https://github.com/ultralytics/yolov5)

| | |
|---|---|
| **Aluno** | Felipe Yamabe Shimoda de Lima |
| **RM** | 573344 |
| **Turma** | 1TIAOB |

---

## Sobre o projeto

A **FarmTech Solutions** passou a oferecer serviços de visão computacional. Em visita a um
cliente, o time precisa demonstrar na prática como um detector de objetos treinado sob medida
funciona, e o quanto ele acerta.

Este repositório reúne as duas entregas obrigatórias do capítulo:

| Entrega | Tema | Pergunta |
|---|---|---|
| **1** | YOLOv5 customizada | Com 80 imagens rotuladas, uma YOLO aprende a detectar maçãs e tesouras? 30 ou 60 épocas? |
| **2** | Abordagens concorrentes | Como a YOLO customizada se compara à YOLO tradicional e a uma CNN treinada do zero? |

---

## O notebook

Todo o passo a passo, o código comentado, as saídas e as análises estão no notebook:

### [FelipeShimoda_rm573344_pbl_fase6.ipynb](FelipeShimoda_rm573344_pbl_fase6.ipynb)

O notebook está com **todas as células executadas** no Google Colab com GPU T4 e pode ser lido
diretamente pelo GitHub. Para executá-lo de novo:

- **[Abrir no Google Colab](https://colab.research.google.com/github/dev-felipeshimoda/Fase-6---Cap-1---O-despertar-da-Rede-Neural/blob/main/FelipeShimoda_rm573344_pbl_fase6.ipynb)**
- **[Abrir no nbviewer](https://nbviewer.org/urls/raw.githubusercontent.com/dev-felipeshimoda/Fase-6---Cap-1---O-despertar-da-Rede-Neural/main/FelipeShimoda_rm573344_pbl_fase6.ipynb)**

No Colab, o notebook conecta o Google Drive, clona este repositório e organiza o dataset no Drive
sozinho. Basta usar **Ambiente de execução > Executar tudo**.

---

## Como a solução foi construída

| Etapa | O que foi feito |
|---|---|
| Objetos | **Maçã** e **tesoura**: bem diferentes entre si e presentes nas 80 classes da base COCO, o que torna justa a comparação com a YOLO tradicional |
| Dataset | 80 imagens, 40 por classe, divididas em 32 para treino, 4 para validação e 4 para teste |
| Rotulação | Caixas delimitadoras desenhadas no [Make Sense](https://www.makesense.ai) e exportadas no formato YOLO |
| Entrega 1 | YOLOv5s treinada a partir dos pesos do COCO, em duas simulações: 30 e 60 épocas |
| Entrega 2 | YOLOv3 original com pesos COCO, sem treino, e CNN em Keras treinada do zero, todas avaliadas nas mesmas 8 imagens de teste |

## Principais resultados

### Entrega 1 | 30 contra 60 épocas

Métricas na validação, na melhor época de cada simulação:

| Métrica | 30 épocas | 60 épocas |
|---|---:|---:|
| Precisão | 0,899 | 0,907 |
| Recall | 0,550 | 0,900 |
| mAP50 | 0,844 | 0,958 |
| mAP50-95 | 0,510 | 0,644 |
| Melhor época | 30 de 30 | 54 de 60 |
| Tempo de treino (GPU T4) | 1,7 min | 2,0 min |

**30 épocas não bastaram.** A melhor época da primeira simulação foi a última, e o modelo
encontrava só 55% dos objetos da validação. Com 60 épocas o recall foi a 0,900 e o mAP50-95, que
exige caixas cada vez mais justas, subiu de 0,510 para 0,644, sem overfitting prejudicial. As 30
épocas extras custaram 21 segundos de GPU.

### Entrega 2 | Três abordagens nas mesmas 8 imagens de teste

| Abordagem | Acurácia por imagem | Precisão | mAP50 | Treino | Inferência CPU | Inferência GPU |
|---|---:|---:|---:|---:|---:|---:|
| YOLOv5s customizada, 30 épocas | 0,750 | **1,000** | **1,000** | 100 s | 186 ms | **15 ms** |
| YOLOv5s customizada, 60 épocas | **1,000** | **1,000** | **1,000** | 121 s | 216 ms | **15 ms** |
| YOLOv3 tradicional (COCO) | **1,000** | 0,732 | 0,784 | **zero** | 1.031 ms | não roda |
| CNN treinada do zero | 0,625 | 0,786 | não se aplica | 11,8 s | **32 ms** | 23 ms |

Três achados que os números sozinhos não contam:

1. **A YOLOv5 de 60 épocas e a YOLOv3 pronta acertaram as 8 imagens, mas não do mesmo jeito.** A
   YOLOv5 não teve nenhum alarme falso; a YOLOv3 marca três maçãs onde existem duas e hesita entre
   `knife` e `scissors` com a mesma confiança. Para dizer "tem maçã aqui" a rede pronta resolve;
   para contar e localizar, não. E ela só funciona porque maçã e tesoura estão entre as 80 classes
   do COCO.
2. **O modelo de 30 épocas viu, mas não confiou.** Ele encontra todos os objetos do teste (mAP50 de
   1,000), mas em duas imagens com confiança abaixo do limiar de 0,25, e as detecções são
   descartadas. Mais treino deixou o modelo mais seguro. Limiar é decisão de negócio, não detalhe
   técnico.
3. **O tipo de rótulo decidiu o resultado.** Mesmas 80 imagens: caixas por objeto levaram a mAP50
   de 1,000 no teste, nenhum rótulo a 0,784 e apenas a classe da imagem a 0,625 de acurácia. A
   rotulação, feita em menos de meia hora, rendeu mais que qualquer ajuste de arquitetura.

Um cuidado na leitura: o teste tem só 8 imagens, então o acerto de 100% não significa um modelo
perfeito. Na validação, o modelo de 60 épocas ainda deixou passar uma das 5 tesouras.

**Recomendação:** YOLOv5 customizada com 60 épocas como base do produto, com o limiar de confiança
ajustável por aplicação.

---

## Demonstração

Esta entrega não inclui vídeo demonstrativo. A demonstração está em dois lugares:

- **[Apresentação do projeto (PDF)](apresentacao-farmtech-vision-fase6.pdf)**, com o problema, o
  método, as imagens de teste processadas e a comparação entre as abordagens.
- **O próprio notebook**, integralmente executado, com cada saída acompanhada da sua análise.

---

## Estrutura do repositório

```
.
├── README.md                                 Este arquivo
├── FelipeShimoda_rm573344_pbl_fase6.ipynb    Notebook com a solução completa, executado
├── apresentacao-farmtech-vision-fase6.pdf    Apresentação do projeto
├── requirements.txt                          Dependências para execução local
└── dataset/
    ├── images/
    │   ├── train/                            64 imagens (32 maçãs, 32 tesouras)
    │   ├── val/                              8 imagens
    │   └── test/                             8 imagens
    └── labels/
        ├── train/                            Rótulos YOLO exportados do Make Sense
        ├── val/
        └── test/
```

## Como executar localmente

```bash
git clone https://github.com/dev-felipeshimoda/Fase-6---Cap-1---O-despertar-da-Rede-Neural.git
```

```bash
pip install -r requirements.txt
```

```bash
jupyter notebook FelipeShimoda_rm573344_pbl_fase6.ipynb
```

O notebook clona o YOLOv5 em um commit fixo e baixa os pesos necessários na primeira execução. Sem
GPU, os treinos levam bem mais tempo do que no Colab.

Dois cuidados:

- **No Colab**, o notebook cria a pasta `MyDrive/FIAP/Fase6_Cap1_FarmTechVision` no Drive de quem
  executa e guarda nela o dataset, os resultados e os pesos da YOLOv3 original (cerca de 250 MB),
  para não baixá-los de novo a cada execução.
- **No Windows**, clone o repositório em uma pasta cujo caminho não tenha acentos: os scripts do
  YOLOv5 não localizam arquivos em caminhos com caracteres acentuados.

## Dataset

As imagens foram reunidas a partir de bases públicas com licença livre (Open Images V7 e Wikimedia
Commons), redimensionadas para no máximo 640 pixels e rotuladas no Make Sense.

---

*Felipe Yamabe Shimoda de Lima | RM 573344 | 1TIAOB | FIAP*
