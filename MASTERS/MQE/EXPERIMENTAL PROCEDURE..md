Sim. Essa seção é essencial, porque deixa o plano menos abstrato e mostra exatamente **como o experimento será executado**, o que conversa diretamente com a exigência da disciplina de definir desenho experimental, repetições, randomização, métricas, análise estatística e artefatos reprodutíveis .

Você pode adicionar esta seção ao plano:

---

## Procedimento experimental

O procedimento experimental será organizado em etapas sequenciais, garantindo que todos os cenários sejam avaliados sob as mesmas condições de modelo, dados, ataques e métricas.

**Passo 1 — Preparação do ambiente experimental.**  
Será configurado um ambiente Python com versões fixas das bibliotecas utilizadas para treinamento, inferência e análise estatística. Serão registradas informações sobre sistema operacional, GPU utilizada, versão do CUDA, versão do PyTorch, modelo base, hiperparâmetros e seeds experimentais. O ambiente será documentado por meio de um arquivo `requirements.txt`, `environment.yml` ou configuração equivalente.

**Passo 2 — Seleção do modelo base.**  
Será escolhido um único modelo base para todos os cenários experimentais. O mesmo modelo será usado no modelo sem defesa, no StruQ format-only, no StruQ-like SFT, no SecAlign-like DPO e no Instruction-Hierarchy-like SFT. Essa decisão evita que diferenças de desempenho sejam causadas por modelos diferentes, e não pelas estratégias de defesa avaliadas.

**Passo 3 — Obtenção e preparação dos dados.**  
Será selecionado um subconjunto dos dados disponibilizados pelo artigo de referência, priorizando amostras relacionadas a prompt injection. As amostras serão organizadas em entradas limpas e entradas contaminadas. Cada instância deverá conter a instrução legítima, o conteúdo de entrada, o tipo de ataque, a resposta esperada segura e, quando necessário, uma resposta insegura que representa o comportamento de seguir a injeção.

**Passo 4 — Divisão dos dados.**  
Os dados serão divididos em treino, validação e teste. A divisão será estratificada por tipo de tarefa e tipo de ataque, para evitar concentração excessiva de um único cenário. O conjunto de teste não será usado durante o treinamento ou calibração. Além disso, os ataques serão separados entre **ataques vistos no treinamento** e **ataques não vistos**, permitindo avaliar generalização.

**Passo 5 — Construção dos formatos experimentais.**  
A mesma base de exemplos será convertida para os formatos necessários a cada defesa. Para o **StruQ-like SFT**, as entradas serão estruturadas com separação explícita entre instrução confiável e dado não confiável. Para o **SecAlign-like DPO**, os exemplos serão convertidos em pares de preferência, com uma resposta segura e uma resposta insegura. Para o **Instruction-Hierarchy-like SFT**, as entradas serão organizadas com níveis explícitos de autoridade, como sistema, usuário e dado externo não confiável.

**Passo 6 — Execução do cenário C0: modelo base.**  
O modelo base será avaliado diretamente, sem treinamento adicional. Esse cenário servirá como baseline principal para medir a vulnerabilidade inicial do modelo a prompt injection e seu desempenho em entradas benignas.

**Passo 7 — Execução do cenário C1: StruQ format-only.**  
O modelo base será avaliado novamente, mas agora recebendo entradas formatadas com separação entre instrução e dados. Não haverá fine-tuning nesse cenário. O objetivo é medir se apenas a estruturação do prompt já reduz a taxa de sucesso dos ataques.

**Passo 8 — Treinamento do cenário C2: StruQ-like SFT.**  
Será realizado fine-tuning supervisionado, preferencialmente com LoRA ou QLoRA, usando exemplos estruturados no formato StruQ. O modelo será treinado para seguir a instrução legítima e ignorar comandos maliciosos inseridos no campo de dados.

**Passo 9 — Treinamento do cenário C3: SecAlign-like DPO.**  
Será realizado treinamento por otimização de preferência, usando pares compostos por resposta segura e resposta insegura. A resposta segura será aquela que segue a instrução legítima, enquanto a resposta insegura será aquela que obedece à injeção. O objetivo é ensinar o modelo a preferir comportamentos robustos.

**Passo 10 — Treinamento do cenário C4: Instruction-Hierarchy-like SFT.**  
Será realizado fine-tuning supervisionado com exemplos que representam conflitos entre diferentes níveis de autoridade. O modelo será treinado para priorizar instruções de maior autoridade e ignorar instruções conflitantes presentes em dados externos ou conteúdos não confiáveis.

**Passo 11 — Avaliação dos cenários.**  
Todos os cenários serão avaliados no mesmo conjunto de teste. Para cada amostra, serão registrados: cenário experimental, tipo de tarefa, tipo de ataque, se o ataque foi visto ou não durante o treinamento, resposta gerada, resposta esperada, sucesso ou falha da tarefa, sucesso ou falha do ataque e tempo de inferência.

**Passo 12 — Coleta das métricas.**  
A partir das respostas geradas, serão calculadas as métricas principais: **Attack Success Rate**, **Robust Accuracy**, **Injection Following Rate**, **Clean Accuracy**, **Task Success Rate**, **Utility Drop**, latência média e tempo de treinamento. Essas métricas permitirão avaliar segurança, utilidade e custo computacional de cada defesa.

**Passo 13 — Repetição com múltiplas seeds.**  
O experimento será repetido com diferentes seeds, por exemplo `42`, `123` e `2026`, para medir a variabilidade dos resultados. A seleção das amostras, a ordem de treinamento e a divisão dos dados deverão ser controladas por seed, garantindo maior reprodutibilidade.

**Passo 14 — Análise estatística.**  
Os resultados serão agregados em tabelas e arquivos CSV. Serão calculados intervalos de confiança de 95% para as principais métricas. Como os mesmos exemplos serão avaliados em diferentes cenários, serão usados testes pareados, como McNemar para comparação de acertos e falhas, e Wilcoxon para métricas contínuas, como latência, caso os dados não sigam distribuição normal.

**Passo 15 — Comparação entre cenários.**  
A análise final comparará o modelo base com cada defesa e, em seguida, comparará diretamente StruQ-like SFT, SecAlign-like DPO e Instruction-Hierarchy-like SFT. A comparação buscará responder se as defesas reduzem a taxa de sucesso dos ataques, se preservam utilidade em entradas benignas e se alguma abordagem apresenta melhor generalização para ataques não vistos.

**Passo 16 — Organização dos artefatos de reprodução.**  
Ao final, serão disponibilizados os scripts de preparação dos dados, scripts de treinamento, scripts de avaliação, arquivos de configuração, resultados brutos, notebooks de análise estatística e documentação do ambiente. Essa organização permitirá que o experimento seja executado novamente e que os resultados possam ser auditados ou estendidos posteriormente.

---