# Aceleração de Correção de Rotação de Imagens (CUDA & OpenMP)

**Status:** Concluído (Projeto Final - MC970 Unicamp)
**Tecnologias:** C, C++, CUDA, OpenMP, Profiling (Nsight Compute, perf)

> **Nota de Integridade Acadêmica:** O código-fonte completo deste projeto é mantido em um repositório privado para respeitar as políticas da Universidade Estadual de Campinas (UNICAMP). Este repositório serve como um estudo de caso (*case study*) apresentando a arquitetura, metodologia e os resultados de otimização alcançados. O relatório técnico completo está disponível em PDF neste repositório.

## O Desafio
O objetivo deste projeto foi paralelizar um algoritmo originalmente escrito em Python para a linguagem C/C++, visando corrigir a inclinação (deskewing) de imagens textuais de forma automática e performática. A correção baseia-se na binarização da imagem, cálculo de projeção horizontal e rotação geométrica para maximizar a uniformidade das linhas.

## Arquitetura e Paralelização

### 1. Aceleração em CPU (OpenMP)
Focamos na paralelização da busca do melhor ângulo de rotação (entre -90º e +90º). 
* Utilizamos pragmas `#pragma omp parallel for` distribuindo iterações entre as threads disponíveis.
* Implementamos regiões críticas para garantir atualizações seguras da melhor solução global, evitando condições de corrida.

### 2. Aceleração Massiva em GPU (CUDA)
Transferimos as operações mais pesadas (memory e compute bound) para a GPU, paralelizando três etapas principais:
* **Binarização:** Mapeamento de 1 thread por pixel (`binarizar_kernel`).
* **Métrica de Avaliação:** Uso avançado de *Shared Memory* e otimização de blocos (configuração 16x16) para contar pixels pretos por linha (`contar_pretos_por_linha`), reduzindo drasticamente o acesso à memória global e melhorando o *Memory Throughput*.
* **Rotação e Interpolação:** Mapeamento em hardware das transformações geométricas (`rotate_binary_kernel` e `rotate_bilinear_kernel`).

## Resultados e Performance (*Speedup*)

Através de um profiling rigoroso utilizando o **NVIDIA Nsight Compute** e **perf**, alcançamos ganhos de performance extremamente significativos em comparação com a execução sequencial original:

* **OpenMP:** Aceleração média e constante de **3x**, independente do tamanho da imagem, demonstrando grande estabilidade na paralelização via CPU.
* **CUDA:** Para imagens de alta resolução (onde o custo de transferência host-device é diluído), atingimos um *speedup* superior a **17x** (redução de mais de 94% no tempo de execução), extraindo o máximo do paralelismo da arquitetura.

---
*Desenvolvido por Lucas Félix e Guilherme Ferreira - IC/UNICAMP (2025)*
