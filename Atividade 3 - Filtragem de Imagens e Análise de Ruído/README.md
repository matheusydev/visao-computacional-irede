# Atividade 3 — Filtragem de Imagens e Análise de Ruído

**Curso:** Visão Computacional — Residência em TIC43
**Professor:** Rafael Carmo
**Unidade 2 | Capítulo 1 | Tarefa 3**

**Notebook (Google Colab):** https://colab.research.google.com/drive/1B_7wCsD3so0247m_bhUhoRaMhIZv82AT?usp=sharing

## Objetivo

Reutilizar o mini dataset construído na Atividade 1 para aplicar ruídos artificiais e filtragem **no domínio da frequência**, observando como os filtros atuam na redução de ruído e na preservação ou no realce de detalhes:

- Inserção de ruído gaussiano
- Inserção de ruído sal e pimenta
- Filtro passa-baixa gaussiano (suavização) em dois níveis: kernels 3×3 e 7×7
- Filtro passa-alta Laplaciano (realce de bordas e detalhes)
- Avaliação do impacto de cada etapa na qualidade visual e em algoritmos clássicos de Visão Computacional (Canny e Otsu)

## Stack utilizada

[![Python](https://img.shields.io/badge/Python-3-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Google Colab](https://img.shields.io/badge/Google_Colab-Notebook-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)](https://colab.research.google.com/)
[![Google Drive](https://img.shields.io/badge/Google_Drive-Storage-4285F4?style=for-the-badge&logo=googledrive&logoColor=white)](https://www.google.com/drive/)
[![OpenCV](https://img.shields.io/badge/OpenCV-cv2-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)](https://opencv.org/)
[![NumPy](https://img.shields.io/badge/NumPy-FFT_e_arrays-013243?style=for-the-badge&logo=numpy&logoColor=white)](https://numpy.org/)
[![pandas](https://img.shields.io/badge/pandas-Tabelas-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualização-11557C?style=for-the-badge&logo=plotly&logoColor=white)](https://matplotlib.org/)
[![Pillow](https://img.shields.io/badge/Pillow-Metadados-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python-pillow.org/)

## Imagens selecionadas

Duas imagens do dataset, uma de cada classe, capturadas no mesmo cenário (objeto centralizado sobre piso de cerâmica claro, vista superior), para que o objeto seja a principal variável entre as classes:

| Classe | Objeto | Arquivo | ISO | Obturador | Abertura |
|---|---|---|---|---|---|
| Classe A | Controle | `img001_original.jpg` | 100 | 1/60 s | f/1.8 |
| Classe B | Nintendo Switch | `img006_original.jpg` | 125 | 1/60 s | f/1.8 |

- **Dispositivo:** Samsung Galaxy A34 5G
- **Resolução original:** 8000×6000 px (48 MP), JPEG convertido do HEIC original
- **Resolução de trabalho:** 1024×768 px, em escala de cinza

## Preparação do ambiente

O notebook monta o Google Drive (`drive.mount`) para acessar o dataset em `/content/drive/MyDrive/dataset` e define uma semente fixa (`SEMENTE = 42`), usada por todas as funções de ruído para garantir resultados reprodutíveis.

- `PIL.Image.open(...)` — lê o formato, o modo de cor, a resolução e a presença de EXIF de cada arquivo (informações que o OpenCV não fornece)
- `cv2.imread` + `cv2.resize(..., interpolation=cv2.INTER_AREA)` — reduz as imagens de 48 MP para 1024×768, mantendo a proporção 4:3. O `INTER_AREA` calcula a média por área e evita serrilhado
- `cv2.cvtColor(..., cv2.COLOR_BGR2GRAY)` — converte para escala de cinza, pois a FFT 2D opera sobre um único canal
- Ficha técnica montada com `pandas.DataFrame`, além de histogramas e medições de apoio (percentis, pixels saturados, ruído residual no fundo, área do objeto)

## Transformações aplicadas

**A. Ruído gaussiano — `ruido_gaussiano()`**
Soma a cada pixel um valor sorteado de uma distribuição normal com `rng.normal(media, sigma, img.shape)` (gerador `np.random.default_rng`), corta o resultado em [0, 255] com `np.clip` e informa a fração de pixels cortados (*clipping*). Parâmetros: **μ = 0 e σ = 25**. A análise usa um mapa do ruído efetivo (ruidosa − original), recortes ampliados e o contraste dos detalhes (Δ/σ) para explicar por que as letras do controle somem e as estampas do Switch resistem.

**B. Ruído sal e pimenta — `sal_pimenta()`**
Sorteia exatamente `densidade × N` posições sem reposição com `rng.choice(img.size, n, replace=False)` e substitui metade por 255 (sal) e metade por 0 (pimenta). Parâmetro: **5% dos pixels**. O notebook registra separadamente a porcentagem de pixels **sorteados** e **efetivamente alterados**, e compara os dois ruídos com `psnr()` e `mae()`.

**C. Impacto em algoritmos — `avaliar_algoritmos()`**
Aplica `cv2.Canny(img, 50, 150)` e a limiarização de Otsu (`cv2.threshold` com `cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU`), contando as regiões segmentadas com `cv2.connectedComponents`. Os mesmos parâmetros são usados antes do ruído, depois do ruído e depois da filtragem, o que permite medir a degradação e a recuperação.

**D. Passa-baixa gaussiano no domínio da frequência — `kernel_gaussiano()`, `mascara_do_kernel()` e `filtrar_frequencia()`**
- `cv2.getGaussianKernel(k, 0)` gera os kernels **3×3** e **7×7** (os mesmos usados pelo `cv2.GaussianBlur`)
- cada kernel é posicionado numa matriz de zeros do tamanho da imagem, com o centro deslocado para (0, 0) via `np.roll`, e `np.fft.fft2` fornece a máscara **H(u,v) exata** do kernel
- a filtragem segue o pipeline `np.fft.fft2` → multiplicação por H → `np.fft.ifft2`, usando a parte real do resultado
- `espectro()` exibe o espectro de magnitude centralizado (`np.fft.fftshift`, em escala `log(1 + |F|)`)

O pipeline é **verificado numericamente**: o resultado na frequência é idêntico à convolução espacial com borda circular (`cv2.filter2D` sobre `np.pad(..., mode='wrap')`) e ao `cv2.GaussianBlur` no interior da imagem. O custo do filtro é medido aplicando-o à imagem sem ruído, com `gradiente_medio()` (Sobel) na região de detalhe.

**E. Passa-alta Laplaciano — `mascara_de()` e `laplaciano()`**
Pela mesma técnica, o kernel Laplaciano de 4 vizinhos (`[[0,1,0],[1,-4,1],[0,1,0]]`) é levado à frequência, gerando uma máscara que vale 0 na frequência zero e −8 nas mais altas. O resultado é verificado contra `cv2.Laplacian(ksize=1)` e exibido de três formas: com sinal (borda dupla), em magnitude e como realce (`f − ∇²f`). A função `regioes_objeto()` (`cv2.connectedComponentsWithStats`, `cv2.findContours` e `cv2.morphologyEx`) divide a imagem em contorno, interior do objeto e fundo, para localizar onde está a resposta mais forte em cada classe.

## Estrutura do notebook

1. **Introdução**
2. **Seleção das imagens** — ficha técnica, condições de captura, histogramas e medições de apoio
3. **Inserção de ruído** — 3.1 gaussiano e 3.2 sal e pimenta
4. **Filtragem passa-baixa gaussiana no domínio da frequência** — conceito, verificação, espectros e máscaras, kernels 3×3 e 7×7, custo do filtro e impacto nos algoritmos
5. **Filtragem passa-alta no domínio da frequência** — Laplaciano, visualização e distribuição da resposta
6. **Comparação final** — figura-resumo, tabela exigida no enunciado, respostas e fechamento

## Estrutura do mini dataset

```
dataset/
├── classe_A/
│   ├── img001_original.jpg   ← utilizada nesta atividade
│   ├── img002_original.jpg
│   ├── img003_original.jpg
│   ├── img004_original.jpg
│   └── img005_original.jpg
└── classe_B/
    ├── img006_original.jpg   ← utilizada nesta atividade
    ├── img007_original.jpg
    ├── img008_original.jpg
    ├── img009_original.jpg
    └── img010_original.jpg
```

## Conclusões gerais

O notebook segue a estrutura Problema → Código → Visualização → Interpretação em cada etapa. Os principais resultados:

- **Qual ruído degrada mais depende do critério:** o gaussiano foi mais prejudicial para a leitura de detalhes e a detecção de bordas (MAE e Canny); o sal e pimenta foi pior em PSNR e na segmentação por Otsu.
- **O kernel 7×7 apresentou o melhor equilíbrio para σ = 25:** reduziu o desvio do ruído de ~25 para ~5 (previsão teórica σ·√Σw² = 5,02) e elevou o PSNR de ~20 para ~33 dB, ao custo de cerca de um terço da nitidez das bordas.
- **O filtro gaussiano não remove o ruído sal e pimenta:** apenas dilui os impulsos em manchas cinzas, o que indica a necessidade de filtragem não linear (mediana).
- **O Laplaciano realça bordas, mas amplifica o ruído:** revelou até a microtextura do piso no reflexo especular, e mostrou que o contraste dos detalhes (letras do controle × estampas do Switch) governa o comportamento de cada classe.
- **Passa-baixa e passa-alta disputam as mesmas altas frequências:** a escolha do filtro e do seu tamanho depende do que a tarefa de Visão Computacional precisa preservar.
