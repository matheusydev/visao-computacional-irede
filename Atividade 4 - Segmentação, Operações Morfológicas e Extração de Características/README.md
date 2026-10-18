# Atividade 4 — Segmentação, Operações Morfológicas e Extração de Características

**Curso:** Visão Computacional — Residência em TIC43
**Professor:** Rafael Carmo
**Unidade 3 | Capítulo 1 | Tarefa 4**

**Notebook (Google Colab):** https://colab.research.google.com/drive/1dlkoc59FaXFjHOKJxhc6Tr3up7XexxTf?usp=sharing

## Objetivo

Aplicar um *pipeline* clássico de Visão Computacional para **contar, delimitar e descrever objetos** em uma imagem, observando como o erro de cada etapa se propaga para a seguinte:

- Segmentação por três métodos: limiarização global (Otsu), limiarização adaptativa e watershed sobre o gradiente de Sobel
- Operações morfológicas: erosão, dilatação, abertura e fechamento
- Identificação dos objetos por componentes conectados
- Extração de nove características de forma, posição, intensidade e textura
- Análise de variabilidade, redundância e confiabilidade das características
- Teste de uma melhoria (correção de iluminação por top-hat) antes de recomendá-la

## Stack utilizada

[![Python](https://img.shields.io/badge/Python-3-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Google Colab](https://img.shields.io/badge/Google_Colab-Notebook-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)](https://colab.research.google.com/)
[![scikit-image](https://img.shields.io/badge/scikit--image-Segmentação_e_GLCM-FF6F00?style=for-the-badge)](https://scikit-image.org/)
[![SciPy](https://img.shields.io/badge/SciPy-Morfologia_binária-8CAAE6?style=for-the-badge&logo=scipy&logoColor=white)](https://scipy.org/)
[![NumPy](https://img.shields.io/badge/NumPy-Arrays-013243?style=for-the-badge&logo=numpy&logoColor=white)](https://numpy.org/)
[![pandas](https://img.shields.io/badge/pandas-Tabelas-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualização-11557C?style=for-the-badge&logo=plotly&logoColor=white)](https://matplotlib.org/)

## Imagem selecionada

Foi usada a imagem `coins` da biblioteca `skimage.data`: moedas gregas antigas sobre fundo escuro, com **múltiplos objetos separados**. Por ser uma imagem padrão da biblioteca, o resultado é reprodutível em qualquer ambiente, sem arquivos externos.

| Informação | Valor |
|---|---|
| Origem | `skimage.data.coins()` |
| Resolução | 384×303 px (116 mil pixels) |
| Tipo | `uint8`, escala de cinza (1 canal) |
| Objetos | 24 moedas em quatro fileiras |
| Característica marcante | Iluminação não uniforme (fundo de ~25 na base a ~107 no canto superior esquerdo) |

## Preparação do ambiente

Bibliotecas importadas: `numpy`, `pandas`, `matplotlib`, `scipy.ndimage` e `skimage` (`data`, `filters`, `morphology`, `measure`, `segmentation`, `color`, `feature`). Três constantes controlam o pipeline: `RAIO_ELEMENTO = 3`, `LIMIAR_FUNDO = 30` e `LIMIAR_MOEDA = 150`.

- `data.coins()` — carrega a imagem, já em escala de cinza
- Ficha técnica montada com `pandas.DataFrame` (resolução, faixa, média, desvio padrão, percentis 1% e 99%, pixels saturados)
- `morphology.opening(img, morphology.disk(35))` — **abertura em tons de cinza** com disco maior que a maior moeda, que apaga as moedas e estima o brilho do fundo por faixas
- As operações morfológicas binárias usam `scipy.ndimage`, e não `skimage.morphology.binary_*`, porque estas estão obsoletas desde a versão 0.26 do scikit-image

## Transformações aplicadas

**A. Segmentação — Otsu, adaptativa e watershed**
- **Otsu global:** `filters.threshold_otsu` escolhe um único limiar T (= 107) que maximiza a variância entre as classes
- **Adaptativa:** `filters.threshold_local` com bloco 101×101 e *offset* −10, comparando cada pixel com a média gaussiana da vizinhança
- **Sobel + watershed:** `filters.sobel` gera o mapa de elevação, os marcadores são definidos onde não há dúvida (fundo < 30, moeda > 150) e `segmentation.watershed` inunda o mapa a partir deles. A função `resumo_mascara()` registra regiões conectadas, regiões < 50 px e a maior região
- Medições de apoio: erro do Otsu por fileira, cobertura dos marcadores por fileira e sensibilidade aos limiares dos marcadores (6 pares testados)

**B. Operações morfológicas — `scipy.ndimage`**
- Preenchimento de furos com `binary_fill_holes`
- Elemento estruturante: disco de raio 3 (`morphology.disk`)
- `binary_erosion` e `binary_dilation` aplicadas isoladamente, apenas para comparação
- Sequência adotada: **abertura → fechamento** (`binary_opening` e `binary_closing`)
- Cada etapa é avaliada pelo número de regiões, pela variação da área total e por um **mapa de diferenças** (verde: pixels adicionados; vermelho: removidos) com ampliação sobre o fragmento espúrio do watershed

**C. Identificação de objetos — `measure.label()`**
- Rotulagem de componentes conectados com vizinhança-8 (`connectivity=2`), comparada com a vizinhança-4
- `measure.regionprops` fornece caixa delimitadora, centroide e demais propriedades de cada moeda, desenhadas sobre a imagem com contorno e identificador

**D. Extração de características — `textura_glcm()` e `regionprops`**
- **Forma:** área, perímetro, circularidade (4π·área/perímetro²), excentricidade e primeiro momento de Hu (φ₁, exibido em escala logarítmica)
- **Posição:** centroide
- **Intensidade:** intensidade média
- **Textura:** contraste e homogeneidade da GLCM (`graycomatrix` e `graycoprops`), calculadas sobre o **quadrado inscrito** na moeda, com 32 níveis de cinza e média de quatro direções (0°, 45°, 90°, 135°) a 1 px de distância
- Estatísticas descritivas com **coeficiente de variação** (CV) e matriz de correlação de Pearson, para medir variabilidade e redundância

**E. Correção de iluminação — transformada top-hat**
- Subtrai da imagem o fundo estimado pela abertura (`f − (f ∘ B)`, disco de raio 35) e repete o Otsu e a morfologia sobre o resultado
- Cada moeda segmentada é comparada à máscara do watershed pelo **IoU** (*Intersection over Union*), considerando bem segmentada quando IoU > 0,9, porque contar regiões não basta para validar uma segmentação

## Estrutura do notebook

1. **Introdução**
2. **Aquisição da imagem** — ficha técnica, visualização, medições de apoio (brilho do fundo) e interpretação com três hipóteses a testar
3. **Segmentação** — conceito, implementação dos três métodos, visualização, medições de apoio e interpretação
4. **Operações morfológicas** — conceito, implementação, visualização com mapas de diferenças e interpretação
5. **Identificação de objetos** — componentes conectados, visualização e possíveis causas de erro
6. **Extração de características** — definição das nove características, implementação e estatísticas descritivas
7. **Análise dos resultados** — variabilidade e redundância, características × posição na imagem, teste da correção de iluminação e interpretação
8. **Conclusão**

## Conclusões gerais

O notebook segue a estrutura Conceito → Implementação → Visualização → Interpretação em cada etapa, e revisa ao final as hipóteses levantadas na Seção 2. Os principais resultados:

| Etapa | Método | Resultado |
|---|---|---|
| Segmentação | Sobel + watershed com marcadores (30 / 150) | 25 regiões (Otsu: 96; adaptativa: 138) |
| Morfologia | Preenchimento → abertura → fechamento, disco de raio 3 | 24 regiões, área total alterada em −0,12% |
| Identificação | Componentes conectados, vizinhança-8 | 24 objetos, sem fusões nem divisões |
| Características | Forma, posição, intensidade e textura (GLCM) | Área e contraste GLCM como as mais informativas |

- **A iluminação não uniforme foi o fator que mais influenciou o trabalho:** inviabilizou o Otsu global (35,8% do fundo da fileira do topo classificado como moeda), tornou as moedas escuras da base as mais frágeis do pipeline e contaminou a intensidade média.
- **O watershed foi o único método utilizável:** decide os pixels ambíguos pela posição das bordas, e não pelo brilho. O resultado é estável numa faixa ampla de marcadores e quebra quando o marcador de moeda desce a 130, abaixo do brilho máximo do fundo.
- **Abertura e fechamento são adequadas para medir; erosão e dilatação isoladas não:** removem o fragmento espúrio alterando a área em menos de 0,2%, enquanto erosão e dilatação alteram cerca de 25%.
- **A contagem correta não garante uma boa segmentação:** o Otsu com morfologia chegou a 25 regiões com uma moeda fundida ao fundo e um fragmento completando a contagem, e só o IoU revelou o erro.
- **Variar não é o mesmo que informar:** área e contraste GLCM são as características mais informativas e complementares (correlação ≈ 0,03). A excentricidade varia por ruído de contorno, a intensidade varia em parte pela iluminação, e circularidade e Hu φ₁ não variam, mas confirmam que os objetos são moedas.
- **A correção de iluminação por top-hat troca um erro por outro:** eliminou a fusão com o fundo, mas compensou em excesso no canto iluminado e deixou as moedas 4 e 5 incompletas. Estimar o fundo com uma superfície polinomial, separar objetos encostados pela transformada de distância e usar LBP como descritor de textura são as melhorias propostas.
