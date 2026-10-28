# Simulação Paralela da Propagação de Incêndio (OpenMP)

Este repositório contém a implementação de um autômato celular bidimensional determinístico para simular a propagação de incêndios florestais. O projeto foi desenvolvido em C com a API OpenMP para explorar o paralelismo em memória compartilhada.

---

## 1. O Modelo Físico e Dinâmica
A simulação modela o avanço do fogo em uma matriz bidimensional com base nos seguintes fatores:
*   **Vizinhança de Moore:** O fogo se propaga para as 8 células adjacentes (ortogonais e diagonais).
*   **Cobertura do Solo e Umidade:** O terreno é classificado em Água, Solo Exposto, Vegetação Rasteira e Floresta, cada um com um fator de combustível e tempo de queima específicos. A umidade local reduz o potencial de ignição recebido pela célula.
*   **Vento:** A direção e a intensidade do vento afetam os pesos direcionais de propagação por meio de um produto escalar (alinhamento).
*   **Zonas de Contenção:** Áreas retangulares programadas para bloquear o avanço do fogo em turnos (passos) específicos, alterando o estado da célula para "Contenção".

---

## 2. Arquitetura e Otimizações Paralelas
Para atingir alto desempenho sem perder o determinismo estrito da simulação, diversas técnicas foram aplicadas na arquitetura do código:

*   **Mapeamento Linear (1D):** A matriz bidimensional é armazenada em vetores unidimensionais contíguos na memória (`indice = linha * C + coluna`) para maximizar a localidade espacial de cache e evitar os atrasos de matrizes dinâmicas tridimensionais.
*   **Double-Buffering:** Utilização de dois espaços de memória paralelos (`estado_atual` e `proximo_estado`) para que as threads leiam do estado atual e escrevam as ignições no próximo simultaneamente, eliminando dependências de leitura e escrita e evitando condições de corrida.
*   **Região Paralela Persistente:** A equipe de threads é criada apenas uma vez no início do laço temporal (`#pragma omp parallel`), eliminando o overhead de criação e destruição a cada passo da simulação.
*   **Vetorização SIMD:** O laço interno de cálculo de transição das células utiliza a diretiva `#pragma omp simd` para processar múltiplas células simultaneamente a nível de instrução vetorial.
*   **Tratamento de Bordas Isolado:** As células localizadas nas extremidades da matriz são processadas separadamente para evitar verificações repetitivas de limites espaciais (branching) dentro do laço principal, tornando o caminho principal mais simples para o compilador.

---

## 3. Compilação e Execução
O projeto inclui um `Makefile` configurado para compilar tanto a versão sequencial (utilizada como base de validação de corretude) quanto a versão paralela.

Para compilar os executáveis:
`make all`

Para executar a simulação passando o arquivo de configuração:
`./fire_seq arquivo_de_entrada.txt`
`./fire_omp arquivo_de_entrada.txt`

---

## 4. Resultados de Desempenho
A versão paralela foi validada numericamente e produz resultados (checksum final, total de ignições, células queimadas) rigorosamente idênticos à versão sequencial. 

Na análise de escalabilidade forte (mantendo o tamanho da matriz fixo), o algoritmo alcançou um **Speedup de ~7.11x** e eficiência superior a 177% operando com 4 threads, superando a barreira linear inicial devido à excelente otimização de localidade de cache, antes de encontrar os limites de largura de banda de memória para contagens maiores de threads.
