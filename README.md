# Projeto de Fine-Tuning: Gerador de Descrição de Produtos com TinyLlama

## Visão Geral do Projeto

Este projeto demonstra um fluxo de trabalho completo para o fine-tuning de um modelo de linguagem de pequeno porte (`TinyLlama-1.1B`) em um ambiente com recursos limitados. O objetivo foi especializar o modelo na tarefa de gerar descrições de produtos no estilo da Amazon, a partir de um título como entrada.

O desafio principal era processar um dataset potencialmente grande e treinar um modelo de forma eficiente, sem incorrer em custos de computação. Para isso, foi empregada uma combinação de técnicas de otimização, como **Quantização em 4-bit** e **Fine-Tuning de Baixa Adaptação de Rank (LoRA)**.

---

## Tecnologias Utilizadas

O sucesso deste projeto se baseia no ecossistema de código aberto da Hugging Face e em bibliotecas de otimização de última geração.

* **Ambiente de Execução: Google Colab**
    * **Descrição**: Plataforma de notebooks baseada em nuvem que oferece acesso gratuito a GPUs (como a NVIDIA T4), tornando-a ideal para prototipagem e execução de projetos de Deep Learning sem a necessidade de hardware local potente.

* **Hugging Face `transformers`**
    * **Descrição**: A biblioteca central do projeto. Ela fornece uma API unificada para baixar, carregar e executar milhares de modelos pré-treinados, incluindo o `TinyLlama`. Foi usada para gerenciar tanto o modelo base quanto o tokenizer.

* **Hugging Face `datasets`**
    * **Descrição**: Ferramenta otimizada para carregar e processar grandes volumes de dados. Foi utilizada para ler o arquivo JSON de produtos e realizar as manipulações de limpeza e formatação de maneira eficiente em memória.

* **Hugging Face `peft` (Parameter-Efficient Fine-Tuning)**
    * **Descrição**: Biblioteca que implementa diversas técnicas para treinar modelos grandes com muito menos recursos computacionais. Em vez de treinar todos os bilhões de parâmetros do modelo, o PEFT permite treinar apenas uma pequena fração deles, economizando memória e tempo.

* **Hugging Face `trl` (Transformer Reinforcement Learning)**
    * **Descrição**: Embora o nome sugira aprendizado por reforço, esta biblioteca oferece ferramentas de alto nível para o treinamento supervisionado, notavelmente o `SFTTrainer` (Supervised Fine-Tuning Trainer). Ele abstrai a complexidade do loop de treinamento, automatizando o processo de alimentação dos dados, cálculo de gradientes e otimização.

* **`bitsandbytes`**
    * **Descrição**: Biblioteca crucial para a otimização de memória. Foi usada para carregar o modelo `TinyLlama` com seus pesos **quantizados em 4-bit**. Isso significa que a precisão numérica dos pesos do modelo foi reduzida, diminuindo drasticamente o consumo de RAM da GPU pela metade, com impacto mínimo na performance.

---

## Metodologia e Técnicas Aplicadas

O projeto seguiu uma metodologia estruturada, aplicando técnicas específicas em cada etapa.

### Etapa 1: Preparação do Dataset

O sucesso do fine-tuning depende criticamente da qualidade dos dados. O processo envolveu:
1.  **Limpeza Profunda**: Remoção de artefatos indesejados do texto, como tags HTML (`<br>`, `<li>`) e entidades HTML (`&#8211;`, `&amp;`), que poderiam confundir o modelo.
2.  **Filtragem de Dados**: Eliminação de entradas duplicadas, nulas ou excessivamente longas para garantir a consistência do dataset.
3.  **Formatação para Instrução (Instruction Tuning)**: Os dados foram reformatados para seguir um padrão de "instrução". Cada exemplo foi transformado na string: `<s>[INST] Gere uma descrição para o seguinte produto: {título} [/INST] {descrição} </s>`. Isso ensina o modelo a se comportar como um assistente que responde a um comando específico, em vez de apenas continuar um texto. Os tokens `<s>` (início), `</s>` (fim), `[INST]` e `[/INST]` são marcadores especiais que definem a estrutura da conversa para o modelo.
4.  **Compatibilidade**: A coluna final foi renomeada para `"text"`, um requisito para garantir a compatibilidade com a versão padrão da biblioteca `trl` no ambiente do Colab.

### Etapa 2: Fine-Tuning de Parâmetros Eficientes (PEFT) com LoRA

Esta é a técnica central que viabilizou o treinamento no Colab.
* **O Problema**: Treinar todos os 1.1 bilhão de parâmetros do TinyLlama exigiria uma quantidade massiva de memória de GPU.
* **A Solução - LoRA (Low-Rank Adaptation)**: Em vez de modificar os pesos originais do modelo, a técnica LoRA congela todos eles e injeta duas matrizes pequenas e "treináveis" (chamadas de adaptadores) em camadas específicas do modelo (geralmente as de atenção). Durante o treinamento, apenas os pesos dessas novas matrizes minúsculas são atualizados.
* **Vantagens**:
    * **Redução Drástica de Memória**: O número de parâmetros treináveis foi reduzido de 1.1 bilhão para apenas alguns milhões.
    * **Portabilidade**: O resultado do treinamento não é um novo modelo de 1.1 bilhão de parâmetros, mas sim um pequeno arquivo (alguns megabytes) contendo apenas os pesos dos adaptadores.
    * **Modularidade**: É possível treinar vários adaptadores LoRA para diferentes tarefas e "plugá-los" no mesmo modelo base conforme a necessidade.

### Etapa 3: Geração de Respostas

Após o treinamento, o modelo foi configurado para uso prático:
1.  **Carregamento do Modelo**: O modelo base `TinyLlama` foi carregado novamente em 4-bit.
2.  **Aplicação dos Adaptadores**: O `PeftModel` foi usado para carregar os adaptadores LoRA salvos e fundi-los de forma eficiente com o modelo base.
3.  **Interface Interativa**: Foi implementado um loop simples que permite ao usuário fornecer títulos de produtos em tempo real e ver a descrição gerada pelo modelo especializado, validando a eficácia do fine-tuning.

---

## Conclusão

Este projeto demonstrou com sucesso que, ao combinar ferramentas modernas do ecossistema Hugging Face e técnicas avançadas de otimização como LoRA e quantização, é totalmente viável especializar modelos de linguagem em tarefas customizadas, mesmo com recursos computacionais limitados. O resultado é um modelo leve, eficiente e altamente capacitado na tarefa para a qual foi treinado.

## Equipe

Este projeto foi desenvolvido por:

- **[Rafael Toccolini](https://www.linkedin.com/in/rafaeltoccolini/)**

- **[Guilherme Santana](https://www.linkedin.com/in/guilherme-santana-04360917a/)**

- **[Franklin Araujo](https://www.linkedin.com/in/franklinarauj/)**

## Link da Apresentação

- https://www.youtube.com/

## Licença

Este projeto está licenciado sob a Licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.