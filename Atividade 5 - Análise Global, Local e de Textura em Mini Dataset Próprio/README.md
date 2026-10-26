# Atividade 5 — Análise Global, Local e de Textura em Mini Dataset Próprio

**Curso:** Visão Computacional — Residência em TIC43
**Professor:** Rafael Carmo
**Unidade 3 | Capítulo 2 | Tarefa 5**

**Notebook (Google Colab):** https://colab.research.google.com/drive/1h8KOpkSLbrCYsxeNeO6gZnvtJze2IpoU?usp=sharing

## Objetivo

Reutilizar o mini dataset construído na Atividade 1 e investigar como **diferentes representações da imagem ajudam — ou falham — na distinção entre classes**, sempre respondendo à pergunta: *o que este descritor preserva e o que ele perde?*

- Nível global: média, variância, desvio padrão, entropia e histogramas de intensidade e de cor (RGB e HSV)
- Comparação entre as imagens originais e as versões derivadas da Atividade 1 (resolução, quantização, JPEG e PNG)
- Nível local: cantos de Shi-Tomasi, blobs por Laplaciano do Gaussiano, descritores ORB e matching com BFMatcher
- Teste de robustez do ORB a rotação, redução de contraste e borramento
- Nível de textura: propriedades da GLCM e histogramas de LBP
- Síntese crítica: qual representação é mais adequada para o dataset e por quê

## Stack utilizada

[![Python](https://img.shields.io/badge/Python-3-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Google Colab](https://img.shields.io/badge/Google_Colab-Notebook-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)](https://colab.research.google.com/)
[![OpenCV](https://img.shields.io/badge/OpenCV-ORB_e_Matching-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)](https://opencv.org/)
[![scikit-image](https://img.shields.io/badge/scikit--image-Blobs,_GLCM_e_LBP-FF6F00?style=for-the-badge)](https://scikit-image.org/)
[![NumPy](https://img.shields.io/badge/NumPy-Arrays-013243?style=for-the-badge&logo=numpy&logoColor=white)](https://numpy.org/)
[![pandas](https://img.shields.io/badge/pandas-Tabelas-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualização-11557C?style=for-the-badge&logo=plotly&logoColor=white)](https://matplotlib.org/)

## Dataset utilizado

Foi usado o **mini dataset próprio da Atividade 1**: duas classes de objetos pretos fotografadas com o mesmo celular. O ponto central do protocolo é que **os cinco cenários de captura foram repetidos para as duas classes**, formando pares de mesmo cenário (img001↔img006, img002↔img007, …). Isso permite separar o efeito da **classe** do efeito do **cenário**.

| Informação | Valor |
|---|---|
| Classe A | Controle (img001 a img005) |
| Classe B | Nintendo Switch (img006 a img010) |
| Dispositivo | Samsung Galaxy A34 5G (HEIC convertido para JPEG) |
| Resolução | 8000×6000 px (48 MP); análises locais e de textura em 1024×768 |
| Formato | JPEG, RGB, 8 bits por canal (0,22 a 1,48 bits/pixel) |
| Cenários | Luz natural, sombra, luz artificial noturna, luz artificial com sombra projetada e luz mista |
| Fundos | Cerâmica clara com veios, piso cinza com pastilhas e tapete com relevo |

```
dataset/
├── classe_A/   img001_original.jpg … img005_original.jpg
└── classe_B/   img006_original.jpg … img010_original.jpg
```

## Preparação do ambiente

Bibliotecas importadas: `os`, `cv2`, `numpy`, `pandas`, `matplotlib` e `skimage.feature` (`blob_log`, `graycomatrix`, `graycoprops`, `local_binary_pattern`). Duas constantes controlam a escala: `TAMANHO_TRABALHO = (1024, 768)` e `ESCALA = 7,8125` (fator entre a resolução original e a de trabalho).

- O notebook monta o Google Drive no Colab e lê a pasta `MyDrive/dataset`; fora do Colab, usa uma pasta `dataset/` local
- As **estatísticas globais e as versões derivadas são calculadas na resolução original** (8000×6000), porque a própria redução de resolução é uma das transformações analisadas
- As análises locais e de textura usam a resolução de trabalho, redimensionada com `cv2.INTER_AREA`, como na Atividade 3
- Para cada imagem foi definida uma **ROI do objeto** (caixa delimitadora), obtida por Otsu + maior componente conexa e ajustada manualmente nas imagens 003, 004 e 008, onde a sombra e as pastilhas confundiram a limiarização

## Transformações aplicadas

**A. Análise global — estatística e histogramas**
- `estatisticas()` calcula média, variância, desvio padrão e **entropia de Shannon** (−Σ pₖ log₂ pₖ, em bits) na imagem inteira, só no objeto e só no fundo
- `separacao()` mede se cada característica separa as classes pela **razão de Fisher**, pela sobreposição das faixas e pela comparação entre a diferença **entre classes (mesmo cenário)** e **entre cenários (mesma classe)**
- Histogramas de intensidade comparando as duas classes no mesmo cenário, e sobreposição das 10 curvas (imagem inteira e só ROI)
- Histogramas RGB e histograma de **matiz (H)** apenas dos pixels coloridos da ROI (S > 60, V > 60), além da fração de pixels **dourados** (H entre 15 e 35)

**B. Relação com a tarefa anterior — versões derivadas**
- Versões regeneradas com os mesmos parâmetros da Atividade 1: resolução 50% e 20%, quantização em 32 e 2 níveis, JPEG q=30 e PNG
- Para cada versão: níveis distintos, estatísticas, tamanho do arquivo e **PSNR**, histogramas sobrepostos e recortes ampliados dos botões do controle e do Joy-Con

**C. Análise local — pontos de interesse, ORB e matching**
- **Shi-Tomasi** (`cv2.goodFeaturesToTrack`, até 300 cantos, `qualityLevel = 0.01`), com contagem de cantos no objeto × no fundo
- **`blob_log`** (σ de 2 a 12, `threshold = 0.2`), com classificação dos blobs que ficam sobre plástico escuro
- **ORB** (`nfeatures = 1500`) em dois pares da mesma classe e um par de classes diferentes, na imagem inteira e restrito à ROI
- **`BFMatcher` com `NORM_HAMMING`** em três níveis: matches brutos (*cross-check*), **teste da razão de Lowe** (0,75) e **inliers geométricos** por homografia com RANSAC
- Matriz de inliers para os **45 pares** de imagens; uma imagem só é associada a outra quando há pelo menos 10 inliers, porque com 4 ou 5 matches o RANSAC sempre encontra uma homografia
- **Teste de robustez:** rotação de 15°, contraste reduzido a 40% e borramento gaussiano (σ = 2); como a transformação é conhecida, cada match é verificado contra a posição verdadeira (erro < 3 px)

**D. Análise de textura — GLCM e LBP**
- Seis recortes de 256×256 px: corpo do controle, Joy-Con estampado (dois por classe) e dois fundos de referência (cerâmica lisa e tapete)
- **GLCM** com 32 níveis de cinza, distâncias 1 e 4 px e quatro ângulos (0°, 45°, 90°, 135°): *contrast*, *homogeneity*, *energy* e *correlation*, além da anisotropia do contraste
- **LBP** com P = 8, R = 1 e método `uniform`: mapa LBP, histograma normalizado de 10 bins e distância **χ²** entre patches
- Matrizes de distância LBP × GLCM, para verificar se as duas técnicas são coerentes

**E. Verificação quantitativa — vizinho mais próximo (1-NN)**
- Cada descritor é usado num classificador 1-NN com validação *leave-one-out*, na imagem inteira e só na ROI
- Para cada erro, verifica-se se o vizinho escolhido foi a foto **da outra classe tirada no mesmo cenário**, para saber se o descritor reconhece o objeto ou a cena

## Estrutura do notebook

1. **Preparação do dataset e hipótese inicial** — árvore de pastas, ficha das imagens, ROIs, fatores de aquisição e hipótese
2. **Análise global** — estatísticas das 10 imagens, separação entre classes, histogramas de intensidade e de cor, e comparação com as versões derivadas
3. **Análise local** — Shi-Tomasi, blobs, ORB, matching, matriz de inliers e teste de robustez
4. **Análise de textura** — seleção dos patches, GLCM, LBP e coerência entre os dois
5. **Síntese integradora** — verificação por 1-NN, tabela comparativa final e conclusão

## Conclusões gerais

Cada etapa do notebook combina implementação, visualização e interpretação apoiada em números calculados, e a conclusão revisa a hipótese levantada na Parte 1. Os principais resultados:

| Descritor | Resultado | Funcionou no dataset? |
|---|---|---|
| Estatística global | Diferença entre classes de 1,2 nível de brilho; entre cenários, 42,4 | Não |
| Histogramas | Intensidade agrupa por cenário; matiz dourado (H ≈ 22) só no Switch | Intensidade não; cor parcialmente |
| ORB + matching | 28,4 inliers médios na mesma classe × 1,0 entre classes; 0 erros no 1-NN | Sim, o melhor |
| GLCM | *Contrast* do Joy-Con ~10× o do controle, mas medindo a estampa | Parcialmente |
| LBP | Distâncias entre classes iguais às distâncias dentro das classes | Não |

- **O fundo foi o fator que mais influenciou o trabalho:** definiu os descritores globais e de textura, concentrou 63% dos keypoints ORB da img004 (nas pastilhas) e gerou inliers falsos entre classes diferentes no rejunte do piso.
- **Estatísticas e histogramas de intensidade descrevem a aquisição, não o objeto:** como os dois objetos são pretos, aplicados à imagem inteira eles acertaram 0 ou 1 de 10 no 1-NN, escolhendo quase sempre a foto do mesmo cenário.
- **A cor foi o melhor sinal global, mas depende de segmentação:** a fração de pixels dourados ficou entre 2,4% e 3,7% em todas as fotos do Switch, mas a sombra quente e o tapete bege também foram contados como "dourado" dentro da ROI retangular do controle.
- **A entropia não mede informação útil:** a redução para 20% da resolução não alterou nenhuma estatística, embora tenha perdido detalhe; a quantização em 32 níveis e o JPEG q=30 reduziram a entropia sem perda visível.
- **Quantidade de matches não é qualidade:** o par de classes diferentes teve mais matches brutos que o par de controles; só o teste da razão e o RANSAC revelaram a diferença.
- **O descritor ORB é robusto, mas o detector é sensível:** 96% a 98% dos matches do Switch continuaram corretos após rotação, redução de contraste e borramento, mas o número de keypoints caiu até 63%. O controle, com detalhes preto sobre preto, perdeu 80% dos keypoints com a redução de contraste.
- **O dataset não é textural:** a textura mais forte está no fundo. A GLCM separou o Joy-Con do corpo do controle pela amplitude do contraste, e o LBP, invariante a essa amplitude, não separou.
- **Repetir os cenários nas duas classes foi decisivo:** com fundos diferentes por classe, histogramas e estatísticas teriam "separado" as classes pelo motivo errado. Para um sistema simples, a recomendação é segmentar o objeto, usar ORB + matching contra imagens de referência e complementar com a fração de pixels dourados.
