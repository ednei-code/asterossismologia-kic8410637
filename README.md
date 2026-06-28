# Asterossismologia da KIC 8410637

> Análise da curva de luz da gigante vermelha KIC 8410637 (missão Kepler) para identificar modos de oscilação estelar via periodograma de Lomb-Scargle. Estimamos ν_max e Δν e inferimos massa, raio e gravidade, comparando com medidas dinâmicas independentes (Frandsen et al. 2013).

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Status](https://img.shields.io/badge/status-concluído-brightgreen)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

## Sumário

- [Sobre o Projeto](#sobre-o-projeto)
- [Pergunta Central](#pergunta-central)
- [Dataset](#dataset)
- [Stack Técnica](#stack-técnica)
- [Pipeline / Metodologia](#pipeline--metodologia)
- [Resultados](#resultados)
- [Estrutura do Repositório](#estrutura-do-repositório)
- [Como Executar](#como-executar)
- [Principais Descobertas](#principais-descobertas)
- [Limitações](#limitações)
- [Referências](#referências)
- [Autor](#autor)

## Sobre o Projeto

Astrossismologia (ou asterossismologia) é o estudo das oscilações naturais de estrelas — pequenas variações periódicas de brilho causadas por ondas acústicas que se propagam pelo interior estelar. Da mesma forma que sismólogos usam terremotos para mapear o interior da Terra, é possível usar essas oscilações ("modos tipo solar") para inferir propriedades físicas internas de uma estrela — densidade, massa, raio e gravidade superficial — sem nunca resolvê-la espacialmente.

Este projeto aplica esse método à **KIC 8410637**, uma gigante vermelha observada pela missão Kepler. O que torna esse alvo particularmente valioso é o fato de ela fazer parte de um **sistema binário eclipsante** (Hekker et al., 2010; Frandsen et al., 2013): isso permite medir massa e raio de forma **independente**, pela dinâmica orbital — criando um teste raro e direto da precisão do método asterossismológico.

## Pergunta Central

> É possível identificar e caracterizar estatisticamente os modos de oscilação estelar da KIC 8410637 a partir da curva de luz medida pelo Kepler, e utilizar esses parâmetros para inferir propriedades físicas do interior da estrela de forma consistente com a literatura (Frandsen et al., 2013)?

## Dataset

| Item | Detalhe |
| --- | --- |
| Fonte | Missão Kepler (NASA), via [MAST](https://archive.stsci.edu/kepler/) / `lightkurve` |
| Alvo | KIC 8410637 (gigante vermelha em binária eclipsante) |
| Pontos brutos | 65.504 |
| Cadência | ~29,4 min (*long cadence*) |
| Linha de base | ~1.470 dias (~4 anos) |
| Período orbital da binária | 408 dias, e = 0,6 (Frandsen et al., 2013) |

## Stack Técnica

- **Linguagem:** Python 3
- **Aquisição/manipulação de dados:** `pandas`, `numpy`
- **Astronomia/séries temporais:** `lightkurve`, `astropy.timeseries.LombScargle`
- **Estatística/processamento de sinal:** `scipy` (`signal`, `ndimage`, `stats`)
- **Visualização:** `matplotlib`, `seaborn`
- **Ambiente:** Google Colab

## Pipeline / Metodologia

| Passo | Etapa | Técnica |
| --- | --- | --- |
| 1 | Setup | Instalação e configuração do ambiente |
| 2 | Aquisição de dados | Leitura da curva de luz (time, flux, flux_err) |
| 3 | Mascaramento de eclipses | Limiar de fluxo + duração mínima + buffer de borda |
| 4 | Estatística do fluxo residual | Comparação de distribuições antes/depois da máscara |
| 5 | Detrending (*flatten*) | `lightkurve.LightCurve.flatten()`, robusto a *gaps* |
| 6 | Análise espectral | Periodograma de Lomb-Scargle + fundo via filtro de mediana (log10) → ν_max |
| 7 | Estimativa de Δν | Autocorrelação do espectro em torno de ν_max |
| 8 | Interpretação física | Relações de escala asterossismológicas → M, R, log g, ρ̄ |

Cada etapa do notebook documenta não só o resultado final, mas também as iterações de depuração (ex.: três versões sucessivas do método de estimativa de fundo espectral no Passo 6, até eliminar artefatos numéricos e de borda) — registradas como parte do processo, não escondidas.

## Resultados

| Parâmetro | Valor Obtido | Literatura / Dinâmico | Diferença |
| --- | --- | --- | --- |
| ν_max | 41,3 µHz | ~45 µHz (Stello et al., reanálise de Frandsen et al. 2013) | ordem de grandeza consistente |
| Δν | 4,63 µHz | 4,65 µHz (relação de escala, guia) | consistente |
| Raio (R) | 10,23 R☉ | 10,74 ± 0,11 R☉ (Frandsen et al., 2013 — dinâmico) | **-4,7%** |
| Massa (M) | 1,26 M☉ | 1,56 ± 0,03 M☉ (Frandsen et al., 2013 — dinâmico) | **-19,3%** |
| log g (cgs) | 2,52 | 2,8 ± 0,3 (Hekker et al., 2010 — preliminar) | dentro da incerteza |
| Densidade média | 0,00165 g/cm³ | — | compatível com gigante vermelha |

A discrepância maior em massa do que em raio **não é um artefato deste pipeline**: Huber (2014), reanalisando essa mesma estrela com um dataset mais longo, reportou um padrão semelhante (~9% em raio, ~17% em massa), atribuído a limitações conhecidas da relação de escala de massa. Reproduzir essa mesma direção e magnitude de discrepância com um método independente e mais simples reforça que o pipeline capturou o sinal físico real da estrela.

## Estrutura do Repositório

```
asterossismologia-kic8410637/
│
├── notebooks/
│   └── asterossismologia.ipynb
│
├── data/
│   └── kic8410637_lc.csv
│
├── outputs/
│   └── figures/
│
├── requirements.txt
├── README.md
└── LICENSE
```

## Como Executar

**Opção 1 — Google Colab (recomendado, sem setup local):**

1. Abra `notebooks/asterossismologia.ipynb` no Colab.
2. Execute as células em ordem — a primeira instala as dependências (`lightkurve`, `astropy`).

**Opção 2 — Ambiente local:**

```bash
git clone https://github.com/<seu-usuario>/asterossismologia-kic8410637.git
cd asterossismologia-kic8410637
pip install -r requirements.txt
jupyter notebook notebooks/asterossismologia.ipynb
```

## Principais Descobertas

- Identificação robusta de 7 eventos de eclipse (4 primários + 3 secundários), com espaçamento consistente com o período orbital conhecido de 408 dias.
- ν_max e Δν estimados por métodos não-paramétricos (busca de picos sobre fundo espectral robusto + autocorrelação), sem recorrer ao ajuste Gaussiano + MCMC padrão da literatura — e ainda assim convergindo para valores fisicamente plausíveis.
- Duas estimativas independentes de Δν (autocorrelação formal e espaçamento observado entre candidatos a ν_max) convergiram para o mesmo valor (~4,6 µHz), reforçando a confiabilidade do resultado.
- Raio inferido por astrossismologia a **-4,7%** do valor medido de forma totalmente independente (dinâmica orbital da binária) — validação direta do método.

## Limitações

- A temperatura efetiva (Teff ≈ 4.680 K) é um valor da literatura (Hekker et al., 2010), não derivado da análise da curva de luz — necessário como entrada para as relações de escala.
- Relações de escala clássicas (Kjeldsen & Bedding, 1995) com referência solar simples, sem correções de metalicidade ou modelagem de grade estelar completa.
- Um trecho residual no excesso de potência normalizado (~20–25 µHz) não foi investigado em profundidade — documentado, não resolvido.
- Estimativa de ν_max e Δν por proeminência de picos, não por ajuste paramétrico formal (Gaussiano + perfis de Harvey via MCMC).

## Referências

- Frandsen, S., et al. (2013). *KIC 8410637: a 408-day period eclipsing binary containing a pulsating giant star.* A&A, 556, A138.
- Hekker, S., et al. (2010). Identificação preliminar da KIC 8410637 como gigante vermelha pulsante em binária eclipsante.
- Huber, D. (2014). Reanálise asterossismológica da KIC 8410637 com dataset Kepler estendido.
- Kjeldsen, H. & Bedding, T. R. (1995). Relações de escala asterossismológicas.
- Huber, D., et al. (2011). Calibração dos parâmetros solares de referência (ν_max,☉, Δν☉).
- Stello, D., et al. *Oscillating red giants in eclipsing binary systems.*

## Autor

**Ednei Cunha Vicente** — Cientista de Dados

Explorando os mistérios do cosmos com Python, Machine Learning e dados reais.

- 💼 LinkedIn: [linkedin.com/in/ednei-cunha-vicente-551b64187](https://www.linkedin.com/in/ednei-cunha-vicente-551b64187/)
- ✍️ Medium: [medium.com/@ednei_vicente](https://medium.com/@ednei_vicente)

Comentários, sugestões e discussões metodológicas são sempre bem-vindos — sinta-se livre para abrir uma *issue* ou me procurar nas redes acima.
