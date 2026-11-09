# Análise Bayesiana com PyMC

Repositório de estudos práticos e teóricos de **Inferência e Modelagem Bayesiana** em Python, baseado na 3ª edição do livro ***Bayesian Analysis with Python: A Practical Guide to probabilistic modeling*** (Osvaldo Martin) e atualizado para as versões modernas do ecossistema probabilístico (**PyMC 6.x**, **ArviZ 1.3+**, **PreliZ**, **Nutpie** e **xarray DataTree**).

* **Repositório Base Oficial (BAP3):** [https://github.com/aloctavodia/BAP3](https://github.com/aloctavodia/BAP3)

---

## Sumário de Tópicos do Estudo (BAP3)

Abaixo estão organizados os capítulos e tópicos fundamentais da obra e do repositório base, relacionando os conceitos estatísticos e os notebooks práticos implementados:

### 1. Pensando Probabilisticamente (*Thinking Probabilistically*)
* **Fundamentos de Inferência Bayesiana:** Interpretação epistemológica da probabilidade, incerteza aleatória vs. epistêmica.
* **Teorema de Bayes:** Atualização sequencial de crenças:
  $$p(\theta \mid Y) = \frac{p(Y \mid \theta)\,p(\theta)}{p(Y)}$$
* **Componentes da Modelagem:**
  * *Priori* $p(\theta)$: Elicitação de conhecimento prévio (informativas, fracamente informativas, conjugadas e de Máxima Entropia).
  * *Verossimilhança* $p(Y \mid \theta)$: Processo gerador dos dados observados.
  * *Posteriori* $p(\theta \mid Y)$ e *Verossimilhança Marginal* (Evidência) $p(Y)$.
* **Resumo da Posteriori:** Estimativas pontuais (média, mediana, moda/MAP) e **Intervalos de Maior Densidade (HDI — *Highest Density Interval*)** vs. intervalos de caudas iguais (ETI).
* **Elicitação de Prioris com `PreliZ`:** Construção interativa e paramétrica de distribuições a partir de restrições de massa de probabilidade (`pz.maxent`, `pz.Beta`, `pz.Exponential`).

### 2. Programando Probabilisticamente (*Programming Probabilistically*)
* **Modelos Discretos — Problema da Moeda (Beta-Binomial / Beta-Bernoulli):**
  * Notebook: [`src/bayesian_analysis/flip-coin.ipynb`](src/bayesian_analysis/flip-coin.ipynb)
  * Elicitação de *priori* via Máxima Entropia (`pz.maxent`), visualização da estrutura em Grafo Acíclico Direcionado (DAG) com `pm.model_to_graphviz`.
  * Amostragem MCMC com **NUTS** via `nutpie`, diagnósticos de convergência ($\hat{R}$, *Effective Sample Size* — ESS, *trace plots* e *rank plots*).
  * Gráficos no estilo de Kruschke (estimativa pontual, HDI de $94\%$ e valor de referência $\theta_{\text{ref}} = 0.5$) e teste de hipóteses via **Fator de Bayes ($BF_{10}$ e $BF_{01}$)** pela razão de Savage-Dickey (`az.plot_bf`).
  * **ROPE (*Region of Practical Equivalence*):** Avaliação de equivalência prática em torno de um valor nulo.
* **Modelos Contínuos — Modelo Gaussiano vs. Modelo Robusto de Student-t:**
  * Notebook: [`src/bayesian_analysis/gaussian-model.ipynb`](src/bayesian_analysis/gaussian-model.ipynb)
  * Inferência sobre média ($\mu$) e escala ($\sigma$) em dados experimentais de deslocamentos químicos de RMN (`chemical_shifts.csv`).
  * **Sensibilidade a *Outliers*:** Demonstração de como as caudas leves da distribuição Normal ($\mathcal{N}(\mu, \sigma)$) inflam $\sigma$ e deslocam $\mu$ na presença de valores extremos.
  * **Regressão/Modelagem Robusta:** Substituição pela distribuição **t de Student** ($\text{StudentT}(\nu, \mu, \sigma)$), estimando o parâmetro de graus de liberdade (normalidade) $\nu \sim \text{Exponential}(1/30)$ para acomodar caudas pesadas.
  * **Checagens Preditivas *a Posteriori* (PPC — *Posterior Predictive Checks*):** Simulação de dados sintéticos $p(\tilde{y} \mid y)$ com `pm.sample_posterior_predictive` e avaliação visual de ajuste com `az.plot_ppc_dist`.
  * **Manipulação de `InferenceData` / `xarray.DataTree`:** Indexação multidimensional (`chain`, `draw`), agregações e extração de amostras com `az.extract`.

### 3. Modelos Hierárquicos (*Hierarchical Models*)
* ** Compartilhamento de Informação (*Partial Pooling*):** Meio-termo estatístico entre modelos completamente agrupados (*pooled*, que ignoram diferenças entre grupos) e modelos independentes (*unpooled*, com alta variância em grupos pequenos).
* **Contração (*Shrinkage*):** Regularização automática das estimativas de grupos em direção à média global da população por meio de hiperprioris.
* **Hierarquias em Dados Binários e Contínuos:** Exemplos clássicos de qualidade da água, deslocamentos químicos por aminoácido e desempenho esportivo.

### 4. Modelando com Retas — Regressão Linear e Generalizada (*Modeling with Lines*)
* **Regressão Linear Simples e Múltipla:** Formulação probabilística $Y \sim \mathcal{N}(\alpha + X\beta, \sigma)$, centralização de preditores para reduzir correlação na posteriori entre intercepto e inclinação.
* **Variância Heterocedástica:** Modelagem do desvio-padrão $\sigma$ como função das variáveis preditoras.
* **Interações e Regressão Polinomial:** Captura de efeitos não aditivos e curvaturas.
* **Modelos Lineares Generalizados (GLMs):** Funções de ligação (*link functions*) para dados não normais:
  * **Regressão Logística** (ligação *logit* + verossimilhança Bernoulli/Binomial para classificação e probabilidades).
  * **Regressão de Poisson e Binomial Negativa** (ligação *log* para dados de contagem e sobredispersão).
  * **Regressão Linear Robusta** (verossimilhança t de Student).

### 5. Comparando Modelos (*Comparing Models*)
* **Trade-off Viés-Variância, Subajuste e Sobreajuste:** Navalha de Ockham na estatística bayesiana.
* **Capacidade Preditiva Fora da Amostra (ELPD — *Expected Log Pointwise Predictive Density*):**
  * **PSIS-LOO-CV (*Pareto Smoothed Importance Sampling Leave-One-Out Cross-Validation*):** Estimativa eficiente de validação cruzada sem reajustar o modelo (`az.loo`).
  * **WAIC (*Widely Applicable Information Criterion*).**
* **Ponderação e Média de Modelos (*Model Averaging*)** e seleção parcimoniosa de variáveis com **Kulprit** (*Projection Predictive Variable Selection*).
* **Fatores de Bayes vs. Critérios Preditivos:** Sensibilidade à *priori* nos Fatores de Bayes em contraste com a estabilidade preditiva do LOO.

### 6. Modelagem de Alto Nível com Bambi (*Modeling with Bambi*)
* **Sintaxe de Fórmulas (*Wilkinson notation*):** Especificação concisa de GLMs e modelos lineares generalizados mistos (GLMMs) sobre o PyMC (`y ~ x1 + x2 + (1|grupo)`).
* **Prioris Padrão Fracamente Informativas:** Escalonamento automático das prioris com base nos dados.
* **Splines e Modelos Aditivos Generalizados (GAMs):** Modelagem flexível de relações não lineares suaves.
* **Modelos Distributivos:** Modelagem simultânea da média e dos parâmetros de dispersão/forma.

### 7. Modelos de Mistura (*Mixture Models*)
* **Misturas Finitas (Gaussian Mixture Models — GMMs):** Modelagem de subpopulações latentes e distribuições multimodais.
* **Não Identificabilidade e *Label Switching*:** Uso de transformações ordenadas (`ordered` transform) para garantir identificabilidade das componentes.
* **Misturas Contínuas:** Modelos *Zero-Inflated* (ex.: Zero-Inflated Poisson) e modelos *Hurdle*.

### 8. Processos Gaussianos (*Gaussian Processes — GPs*)
* **Modelagem Não Paramétrica sobre Funções:** Prioris sobre espaços de funções $f(x) \sim \mathcal{GP}(m(x), k(x, x'))$.
* **Funções de Covariância (*Kernels*):** *Exponentiated Quadratic* (RBF), Matérn, periódicos e combinações lineares/multiplicativas de kernels.
* **Aproximações Escaláveis:** **HSGP (*Hilbert Space Gaussian Processes*)** para grandes volumes de dados e GPs com verossimilhanças não gaussianas (classificação espacial e contagens).

### 9. Árvores de Regressão Aditivas Bayesianas (*Bayesian Additive Regression Trees — BART*)
* **Soma de Árvores com `PyMC-BART`:** Combinação não paramétrica de múltiplas árvores de decisão rasas regularizadas por prioris que evitam *overfitting*.
* **Detecção Automática de Não Linearidades e Interações:** Estimativa de dependência parcial (*Partial Dependence Plots* — PDPs), efeitos individuais (*ICE plots*) e importância de variáveis.

### 10. Motores de Inferência (*Inference Engines*)
* **Métodos Não Markovianos:** Aproximação em Grade (*Grid Approximation*), Aproximação Quadrática/Laplace e **Inferência Variacional (ADVI)**.
* **Métodos MCMC (*Markov Chain Monte Carlo*):**
  * Metropolis-Hastings e amostragem de Gibbs.
  * **Hamiltonian Monte Carlo (HMC)** e **NUTS (*No-U-Turn Sampler*)**: Uso de gradientes da log-posteriori para exploração eficiente em alta dimensão.
  * **Sequential Monte Carlo (SMC):** Amostragem para posterioris multimodais e cálculo direto de evidência marginal.
* **Diagnósticos de Convergência e Geometria:** Divergências no espaço de fase, *Energy Plots* (BFMI), $\hat{R}$ (vehtari), ESS (*bulk* e *tail*) e autocorrelação.

### 11. Próximos Passos (*Where to Go Next*)
* Inferência Causal (DAGs causais, *do-calculus*, variáveis confundidoras, mediadores e colisores), séries temporais bayesianas, análise de sobrevivência e tomada de decisão sob incerteza (funções de perda).

---

## Notebooks Disponíveis no Projeto

| Notebook | Capítulo (BAP3) | Principais Conceitos e Ferramentas |
| :--- | :---: | :--- |
| [`flip-coin.ipynb`](src/bayesian_analysis/flip-coin.ipynb) | Cap. 1 e 2 | Priori Beta via `pz.maxent`, modelo Bernoulli em PyMC, amostragem com `nutpie`, gráfico estilo Kruschke (HDI + valor de referência), Fator de Bayes (`az.plot_bf`), ROPE. |
| [`gaussian-model.ipynb`](src/bayesian_analysis/gaussian-model.ipynb) | Cap. 2 | Modelo Normal vs. Modelo Robusto t de Student (`chemical_shifts.csv`), tratamento de *outliers*, `az.plot_pair` conjunto + marginais, PPC (`az.plot_ppc_dist`), manipulação de `xarray.DataTree`. |

---

## Guia Rápido de Migração: `ArviZ < 1.0` (Livro) $\rightarrow$ `ArviZ 1.3+` (Este Repositório)

O código original do livro utiliza `ArviZ 0.x`. Neste projeto utilizamos a nova arquitetura **ArviZ 1.3+ (`arviz-plots` / `arviz-stats` / `xarray.DataTree`)**. As principais equivalências adotadas nos notebooks são:

| Tarefa | Código Antigo (`ArviZ < 1.0`) | Código Atual (`ArviZ >= 1.0`) |
| :--- | :--- | :--- |
| **Configurar HDI padrão** | Padrão implícito em funções | `az.rcParams['stats.ci_kind'] = 'hdi'`<br>`az.rcParams['stats.ci_prob'] = 0.94`<br>`az.rcParams['stats.point_estimate'] = 'mean'` |
| **Gráfico da Posteriori (Kruschke)** | `az.plot_posterior(idata, ref_val=0.5)` | `az.plot_dist(idata)` combinado com `az.add_lines` e `arviz_plots.visuals.annotate_xy` |
| **Gráfico Conjunto (Pair Plot)** | `az.plot_pair(idata, kind='kde', marginals=True)` | `az.plot_pair(idata, marginal=True, marginal_kind='kde', visuals={'contour': True, 'scatter': False})` |
| **Posterior Predictive Check (PPC)** | `az.plot_ppc(idata, num_pp_samples=100)` | `az.plot_ppc_dist(idata, num_samples=100)` |
| **Adicionar grupo `prior` ao `idata`** | `idata.extend(...)` ou dicionário | `idata.update({'prior': pm.convert_to_datatree(...)['posterior']})` |

---

## Stack Tecnológico e Instalação

O projeto é gerenciado com [**uv**](https://docs.astral.sh/uv/) e requer **Python $\ge 3.12$**.

### Principais Dependências (`pyproject.toml`)
* **Modelagem Probabilística e Amostragem:** `pymc >= 6.3.1`, `nutpie >= 0.16.11`, `bambi >= 0.20.0`, `pymc-bart >= 0.13.1`
* **Diagnósticos, Visualização e Seleção:** `arviz >= 1.3.0`, `preliz >= 0.28.0`, `kulprit >= 0.6.1`, `matplotlib >= 3.11.1`
* **Computação Científica e Estruturas de Dados:** `numpy >= 2.5.3`, `scipy >= 1.18.1`, `pandas >= 3.0.5`, `xarray >= 2026.7.0`, `numba >= 0.67.0`

### Como Executar

1. **Sincronizar o ambiente virtual com o `uv`:**
   ```powershell
   uv sync
   ```

2. **Instalar o Graphviz (necessário para `pm.model_to_graphviz`):**
   ```powershell
   winget install Graphviz.Graphviz
   ```
   *(Caso o executável `dot.exe` não esteja no `PATH` da sessão atual, adicione `C:\Program Files\Graphviz\bin` ao `PATH`).*
