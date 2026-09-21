# Atividade 2 — Suavização, Remoção de Ruído e Detecção de Bordas

**Curso:** Visão Computacional — Residência em TIC43
**Professor:** Rafael Carmo
**Unidade 2 | Capítulo 1 | Tarefa 1**

**Notebook (Google Colab):** https://colab.research.google.com/drive/1Mmdb5uxCBdv4UcXfAt-LxP4AHcdnA_nx?usp=sharing

## Objetivo

Aplicar filtros de suavização e operadores de detecção de bordas em 4 imagens do mini dataset construído na Atividade 1, avaliando qual filtro é mais adequado ao tipo de degradação presente e se a suavização prévia melhora a detecção de bordas:

- Filtros de suavização: média, Gaussiano e mediana
- Operadores de borda: Sobel, Prewitt e Canny
- Análise quantitativa (média, variância e contagem de pixels de borda) separada da observação visual
- Efeito da suavização prévia sobre a detecção de bordas

## Stack utilizada

[![Python](https://img.shields.io/badge/Python-3-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Google Colab](https://img.shields.io/badge/Google_Colab-Notebook-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)](https://colab.research.google.com/)
[![Google Drive](https://img.shields.io/badge/Google_Drive-Storage-4285F4?style=for-the-badge&logo=googledrive&logoColor=white)](https://www.google.com/drive/)
[![OpenCV](https://img.shields.io/badge/OpenCV-cv2-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)](https://opencv.org/)
[![NumPy](https://img.shields.io/badge/NumPy-Array_ops-013243?style=for-the-badge&logo=numpy&logoColor=white)](https://numpy.org/)
[![Pandas](https://img.shields.io/badge/Pandas-Tabelas-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualização-11557C?style=for-the-badge&logo=plotly&logoColor=white)](https://matplotlib.org/)

## Dataset utilizado

O dataset é o mesmo da Atividade 1: 10 imagens em duas classes, capturadas com um Samsung A34 5G sob cinco cenários de iluminação replicados nas duas classes.

| Classe | Objeto | Imagens |
|---|---|---|
| Classe A | Controle | `img001` a `img005` |
| Classe B | Nintendo Switch | `img006` a `img010` |

## Preparação do ambiente

O notebook monta o Google Drive (`drive.mount`) para acessar o dataset em `/content/drive/MyDrive/dataset` e configura o Matplotlib com `interpolation='nearest'`, evitando que a própria exibição suavize as imagens e distorça a análise visual.

## Parte 1 — Seleção das imagens

Foram escolhidos os cenários 3 (luz artificial noturna, distância longa) e 5 (luz mista) nas duas classes, por serem os de menor disponibilidade de luz. Usar os mesmos cenários em A e B mantém a condição de captura controlada na comparação entre classes.

| Imagem | Classe | Parâmetros | Degradação observada |
|---|---|---|---|
| `img003_original.jpg` | A (Controle) | ISO 4000, 1/4 s, ƒ/1.8 | Granulação fina no fundo de cerâmica e perda de microtextura no plástico |
| `img005_original.jpg` | A (Controle) | ISO 800, 1/20 s, ƒ/1.8 | Ruído de luminância baixo e perda de definição no tecido de fundo |
| `img008_original.jpg` | B (Nintendo Switch) | ISO 4000, 1/4 s, ƒ/1.8 | Granulação fina, mais perceptível na tela escura do console |
| `img010_original.jpg` | B (Nintendo Switch) | ISO 800, 1/20 s, ƒ/1.8 | Ruído baixo e reflexo especular ocupando a tela do console |

**Carregamento e ROI.** As imagens originais têm 8000×6000 px (48 MP). Cada uma é lida em escala de cinza (`cv2.IMREAD_GRAYSCALE`) e **recortada** em uma região de interesse que contém o objeto inteiro e parte do fundo. O recorte é intencional: redimensionar calcula médias de pixels vizinhos, ou seja, é uma suavização, e alteraria o ruído antes do experimento.

**Linha de base (média e variância por imagem):**

| Imagem | Cenário | Média | Variância | Desvio-padrão |
|---|---|---|---|---|
| img003 | Luz artificial noturna | 91,34 | 1996,9 | 44,7 |
| img005 | Luz mista | 92,40 | 3376,3 | 58,1 |
| img008 | Luz artificial noturna | 82,33 | 2084,9 | 45,7 |
| img010 | Luz mista | 90,59 | 3162,7 | 56,2 |

A variância global é dominada pelo contraste estrutural da cena (objeto escuro sobre fundo claro), e não pelo ruído. Isso gerou uma previsão testada na Parte 2: os filtros devem reduzir pouco a variância global.

## Parte 2 — Filtros de suavização

**A. Filtro da média — `cv2.blur(img, (k, k))`**
Todos os vizinhos do kernel têm o mesmo peso. Testado com kernels 3×3 e 5×5.

**B. Filtro Gaussiano — `cv2.GaussianBlur(img, (0, 0), sigma)`**
Média ponderada pela distância. Com `(0, 0)`, o OpenCV calcula o tamanho do kernel a partir de σ (≈ 6σ+1), o que mantém o filtro fiel ao σ declarado. Testado com σ=1 e σ=3.

**C. Filtro da mediana — `cv2.medianBlur(img, k)`**
Filtro não linear que escolhe o valor central da vizinhança ordenada, sem criar tons intermediários. Testado com kernels 3×3 e 5×5.

**D. Comparação estatística e inspeção 1:1**
Como as figuras completas são exibidas reduzidas e escondem o efeito dos filtros, a comparação usa:
- Tabela de redução percentual da variância global por filtro (`pandas`)
- Painéis 2×4 em escala real (1:1) com recorte de 200×200 px em região homogênea, medindo a variância local (ruído)
- Painéis 2×4 em escala real com recorte de 300×300 px em região de borda, medindo a energia de borda (magnitude média do Sobel)

**Resultados:**
- **Maior redução de ruído:** Gaussiano σ=3, com queda de 55% a 67% na variância local da região homogênea
- **Melhor preservação de bordas:** Mediana 3×3, que mantém contornos e serigrafias afiados; Gaussiano σ=3 e Média 5×5 borram visivelmente as transições
- **Variância global:** redução de no máximo 4,5% para qualquer filtro, contra até 67% no recorte homogêneo, confirmando a previsão da Parte 1

## Parte 3 — Detecção de bordas (sem suavização)

Os operadores são aplicados diretamente nas imagens recortadas:

- **Sobel** — `cv2.Sobel(img, cv2.CV_32F, ...)`: kernel com peso 2 na linha central, que suaviza levemente na direção perpendicular
- **Prewitt** — `cv2.filter2D` com kernels próprios: mesmo princípio do Sobel, com pesos uniformes
- **Canny** — `cv2.Canny` com três pares de limiares (50/150, 100/200 e 150/300): suavização gaussiana interna, supressão não-máxima e histerese

Para Sobel e Prewitt, um pixel é borda quando a magnitude ultrapassa um **limiar fixo (50)**, igual para todas as imagens. A comparação usa a **porcentagem** de pixels de borda, já que as ROIs têm tamanhos diferentes. O uso de `CV_32F` preserva os gradientes negativos, que seriam truncados em `uint8`.

| Imagem | Sobel | Prewitt | Canny (100/200) |
|---|---|---|---|
| img003 (cerâmica, ISO 4000) | 0,41% | 0,22% | 0,03% |
| img005 (tecido, ISO 800) | 3,78% | 2,20% | 0,33% |
| img008 (cerâmica, ISO 4000) | 0,62% | 0,24% | 0,01% |
| img010 (tecido, ISO 800) | 10,82% | 8,04% | 1,74% |

**Observações:**
- As maiores contagens estão nas imagens sobre tecido, e não nas de ISO 4000: o que eleva a contagem é a textura real do fundo, que gera bordas legítimas para o gradiente, mas espúrias para a tarefa
- Na `img010`, o reflexo na tela é detectado pelos três operadores, mas pertence à cena refletida e não ao Nintendo Switch, e nenhum operador baseado em gradiente distingue isso
- **Canny** apresentou menos bordas falsas e bordas finas de cerca de 1 pixel; Sobel e Prewitt geram bordas mais espessas e sensíveis à textura

## Parte 4 — Detecção de bordas após suavização

- **Filtro escolhido:** Gaussiano (`Gauss σ=1`), adequado à granulação fina de alta frequência dos cenários de baixa luz (ruído de leitura e *shot noise* do ISO 4000), por atenuar o ruído sem deslocar as bordas reais como o filtro da média
- Sobel, Prewitt e Canny são reaplicados com **os mesmos parâmetros** da Parte 3, para que qualquer diferença seja atribuída apenas à suavização
- Comparação antes/depois em painéis 2×3 e em tabela com a porcentagem de pixels de borda e a variação percentual por operador e imagem

## Relatório final

- **Quantidade de bordas:** redução no volume de pixels classificados como borda após a suavização
- **Bordas falsas:** atenuação dos picos isolados de variância no fundo e em superfícies uniformes
- **Continuidade das bordas reais:** melhor conexão de segmentos de contorno, especialmente com a histerese do Canny
- **A suavização melhorou a detecção?** Sim
- **Operador mais robusto ao ruído:** Canny; entre os diretos, Sobel foi mais robusto que o Prewitt (pesos `[1, 2, 1]`)
- **Melhor definição estrutural:** Canny, com bordas finas e contínuas

## Organização do notebook

```
Atividade 2
├── Introdução e bibliotecas
├── Parte 1: Seleção das imagens, ROI e linha de base
├── Parte 2: Filtros de suavização (média, Gaussiano, mediana)
├── Parte 3: Detecção de bordas sem suavização (Sobel, Prewitt, Canny)
├── Parte 4: Detecção de bordas após suavização
├── Relatório final (antes/depois e síntese)
└── Conclusão
```

## Conclusões gerais

O conflito central é que o ruído de alta frequência e as bordas dos objetos são ambos variações abruptas de intensidade. A suavização resolve o problema do ruído, mas atenua a nitidez das bordas, exigindo equilíbrio entre o tamanho do kernel e a preservação de detalhes.

Limitações identificadas:
1. **Variância global:** não distingue variação de brilho causada por ruído de variação estrutural legítima da cena
2. **Dependência de limiares:** a contagem de bordas é muito sensível aos limiares escolhidos
3. **Pré-processamento de hardware:** as imagens já passaram por redução de ruído e realce de nitidez do ISP da câmera, o que altera a distribuição estatística do ruído do sensor

Para um pipeline de classificação automática, a remoção do ruído evita descritores de forma espúrios e permite que características geométricas (como Joy-Cons, proporção de tela e botões analógicos) sejam capturadas com mais precisão.