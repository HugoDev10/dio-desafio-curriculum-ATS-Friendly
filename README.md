# 📋 APP Gerador de Currículo ATS Friendly com Vibe Coding 

Desafio DIO (Junto com Santander) criando um gerador de Currículos ATS Friendly.

# CurriculoMatch

O **CurriculoMatch** é uma aplicação web inteligente que analisa a compatibilidade entre o currículo do candidato e a descrição de uma vaga de emprego, calculando uma pontuação de aderência (*Match Score*), identificando lacunas e gerando uma versão otimizada para sistemas ATS (*Applicant Tracking Systems*), sem nunca inventar experiências falsas.

---

### 1. Qual problema a sua aplicação resolve?

Milhões de currículos são rejeitados automaticamente antes mesmo de um recrutador humano lê-los. Os sistemas de triagem automatizada (ATS) descartam candidaturas por:
- **Incompatibilidade de palavras-chave:** O candidato possui a competência, mas usou termos diferentes dos exigidos na vaga.
- **Falha de leitura e formatação:** Layouts complexos (duas colunas, gráficos, tabelas, ícones ou caixas de texto) quebram os *parsers* automáticos dos robôs.
- **Falta de clareza nas lacunas:** O candidato não sabe exatamente quais requisitos da vaga faltam em seu perfil para se preparar.

O **CurriculoMatch** resolve isso em poucos segundos e sem necessidade de cadastro: extrai o texto do currículo do candidato, cruza com a descrição da vaga, entrega um diagnóstico detalhado (Match Score, competências presentes vs. ausentes e recomendações) e gera um currículo com estrutura padrão ATS, pronto para download em PDF.

---

### 2. O mega prompt que você usou, e o que mudou nele até a versão final?

O ponto de partida foi um **mega prompt de especificação de produto (PRD)** que estabeleceu a premissa fundamental do app: **ética e verdade**. A IA foi estritamente instruída a nunca criar dados falsos.

PRD (Product requirement document) refinado no Copilot Web: 

```markdown
# MATCHATS MVP

Crie uma aplicação web chamada **MatchATS** para ajudar candidatos a adaptar seus currículos para vagas específicas usando IA.

## Stack

- React
- TypeScript
- TailwindCSS
- ShadCN/UI
- Supabase

## Design

Utilizar ShadCN/UI em toda aplicação.

### Cores

- Branco (#FFFFFF)
- Verde água (#2DD4BF)
- Tons neutros Slate

### Estilo

- Minimalista
- Moderno
- Limpo
- Responsivo
- Mobile First

---

# Objetivo

O usuário deve conseguir:

1. Enviar seu currículo.
2. Colar a descrição de uma vaga.
3. Receber uma análise de compatibilidade.
4. Gerar uma versão ATS-Friendly do currículo.
5. Exportar o resultado em PDF.

---

# Fluxo Principal

## Tela Única

### Upload de Currículo

Aceitar:

- PDF
- DOCX

### Campo da Vaga

Textarea para colar a descrição da vaga.

### Botão

"Analisar Compatibilidade"

---

# Resultado da Análise

Exibir:

## Match Score

Pontuação de 0 a 100.

## Skills Encontradas

Lista de competências encontradas na vaga e no currículo.

## Skills Ausentes

Lista de competências presentes na vaga mas ausentes no currículo.

## Recomendações

Sugestões objetivas para aumentar a compatibilidade.

---

# ATS Generator

Botão:

"Gerar Currículo ATS"

A IA deve:

- Manter apenas informações verdadeiras do currículo original.
- Não inventar experiências.
- Não inventar certificações.
- Reorganizar o conteúdo.
- Melhorar a redação.
- Destacar palavras-chave relevantes da vaga.
- Produzir uma versão ATS-Friendly otimizada.

---

# Exportação

Permitir download em:

- PDF

---

# Componentes ShadCN

Utilizar:

- Card
- Button
- Input
- Textarea
- Progress
- Badge
- Alert
- Dialog
- Toast

---

# Experiência

- Drag and Drop para upload.
- Loading states.
- Skeleton loaders.
- Mensagens de sucesso e erro.
- Interface simples e extremamente rápida.

---

# Escopo MVP

Não implementar:

- Login
- Cadastro
- Planos
- Pagamentos
- Dashboard complexo
- Histórico
- Multiusuários
- Cover Letter
- Analytics
- Integrações externas

Foco total em um único fluxo:

**Currículo → Vaga → Match Score → Currículo ATS → Download PDF**

```

Para a execução da IA, o mega prompt foi dividido em duas funções com **Structured Outputs (JSON Schema via Zod)**:

1. **Prompt de Análise (`ANALYSIS_INSTRUCTIONS`):**
   - Atua como especialista em recrutamento e sistemas ATS.
   - Gera o `matchScore` (0 a 100), `summary`, `matchedSkills`, `missingSkills` e recomendações acionáveis.
   - **Regra de ouro:** Basear-se exclusivamente no texto fornecido e nunca sugerir inventar qualificações.

2. **Prompt de Otimização ATS (`ATS_INSTRUCTIONS`):**
   - Atua como redator técnico de currículos.
   - Reorganiza as seções em ordem linear recomendada para ATS, aprimora a redação com verbos de ação e destaca palavras-chave da vaga que já existam no currículo original.

#### O que mudou até a versão final:
- **Imposição de tipagem rígida (Zod Schema):** A IA deixou de responder texto livre ou JSON solto; passou a retornar estruturas validadas com campos obrigatórios e tratamento de nulos.
- **Blindagem contra alucinações:** Inclusão de regra expressa: *"Se uma competência da vaga não existir no currículo original, NÃO a inclua no currículo otimizado."*
- **Formatação amigável para parseadores:** Proibição explícita de emojis, tabelas ou formatação markdown interna que pudesse poluir o arquivo final.

---

### 3. Como a análise funciona, da vaga colada até o currículo ajustado?

O fluxo ocorre em 4 etapas lineares:

```text
[Upload PDF/DOCX] ──> [Extração no Navegador] ──> [Server Function + IA] ──> [Score + PDF ATS]
```

1. Extração Local no Navegador (Client-side):
O arquivo (PDF via pdfjs-dist ou DOCX via mammoth) é lido diretamente na máquina do usuário. O arquivo bruto nunca é enviado para servidores nem armazenado em disco, preservando a privacidade e agilizando o processamento.
2. Processamento no Servidor (Server Function):
O texto puro do currículo e o texto da vaga são enviados para rotas seguras de servidor (createServerFn no TanStack Start).
3. Análise de IA com Saída Estruturada:
A IA processa o cruzamento das informações utilizando o Lovable AI Gateway, retornando o diagnóstico completo e o modelo JSON do currículo adaptado.
4. Visualização e Download do PDF ATS:
O frontend renderiza na tela a prévia fiel e compila o PDF no próprio navegador via jspdf, em uma única coluna com tipografia limpa, hierarquia clara e leitura perfeita para qualquer sistema ATS.

### 4. Que ajustes você pediu depois da primeira geração, e por quê?
- Auditoria de Segurança de Credenciais: Varredura completa para assegurar que nenhuma chave de API, segredo ou token JWT ficasse versionado no código-fonte público.
- Mudança de Nome e Posicionamento: O projeto iniciou como MatchATS e foi renomeado para CurriculoMatch, um nome mais intuitivo, memorável e direcionado ao mercado brasileiro.
- Criação de Identidade Visual e Logo Tecnológica: Desenvolvimento de uma logo moderna com símbolo de aprovação de currículo, além de favicon dedicado e metadados de compartilhamento (Open Graph / Twitter Cards).
- Paridade entre Visualização e Impressão: Ajuste fino para garantir que a ordem das seções exibida na tela (AtsResumeView) fosse idêntica à ordem gerada no documento PDF baixado.

### 5. O endereço da aplicação publicada
https://curriculomatch.lovable.app/

## 📸 Telas da Aplicação

### Tela Inicial — Upload do Currículo e Descrição da Vaga
<img width="1912" height="958" alt="image" src="https://github.com/user-attachments/assets/0505f969-ba4d-4d90-bbb2-4ccd53435b0b" />
Visão geral da página inicial do CurriculoMatch com o cabeçalho exibindo a logo e o badge "Grátis · Sem cadastro". No centro, o título de impacto e a linha de progresso das etapas (Currículo → Vaga → Match Score → Currículo ATS → PDF). À esquerda, o card com a área de upload arrastável (dropzone) para arquivos PDF ou DOCX e o campo de texto para colar os requisitos da vaga com contador de caracteres. À direita, a área inicial vazia (Empty State) com ilustração e orientações convidativas.

---

### Diagnóstico de Compatibilidade — Match Score e Análise de Requisitos
<img width="1917" height="961" alt="image" src="https://github.com/user-attachments/assets/06d490eb-6229-443e-abab-e0ae5b09fc39" />
Painel de resultados após a análise da IA. No topo, o indicador de destaque com a pontuação de aderência (Match Score de 0 a 100), acompanhado de um resumo analítico sobre o perfil do candidato frente à vaga. Abaixo, duas colunas visuais contrastantes: competências encontradas (Matched Skills, em verde) e competências ausentes (Missing Skills, em alerta), seguidas por uma lista de recomendações práticas para melhorar a candidatura.

---

### Geração do Currículo Otimizado — Estrutura ATS-Friendly
<img width="642" height="249" alt="image" src="https://github.com/user-attachments/assets/77ab07cb-bebf-4b3f-90eb-77a80dc6fe93" />

Card de geração do currículo adaptado para robôs de triagem. Exibe o selo de transparência e ética ("100% baseado no seu currículo — A IA não inventa experiências nem certificações"). Logo abaixo, apertando o botão Gerar Currículo ATS, aparecerá a prévia do currículo reformatado em coluna única e tipografia limpa, destacando palavras-chave da vaga já existentes na trajetória do profissional, acompanhado dos botões de ação: "Baixar PDF"*, "Ver completo" e "Gerar novamente".

---

### Modal de Revisão Completa e Exportação em PDF
<img width="648" height="634" alt="image" src="https://github.com/user-attachments/assets/210fedc9-2129-4670-9abb-a9ea290163ad" />

Janela modal em primeiro plano com rolagem independente, permitindo ao candidato inspecionar o currículo completo antes do envio: dados de contato, resumo profissional, habilidades-chave, experiências com verbos de ação e formação acadêmica. No rodapé da modal, o botão de destaque para download imediato do arquivo `.pdf` gerado no próprio navegador.


## 💭 Reflexão Final e Aprendizados


O **CurriculoMatch** nasceu da inquietação diante de um dos maiores gargalos do mercado de trabalho moderno: profissionais talentosos sendo sumariamente descartados por filtros algorítmicos (ATS), muitas vezes não por falta de competência, mas por pequenos desencontros de vocabulário, estruturas gráficas ilegíveis para robôs ou falta de clareza na apresentação de seus resultados.

Construir essa solução trouxe aprendizados valiosos em três pilares fundamentais:

### Ética e Confiabilidade na Inteligência Artificial
A tentação comum em ferramentas de IA generativa é prometer um "currículo perfeito" à custa de alucinações — criando habilidades fictícias ou inflando experiências para forçar um resultado 100%. No CurriculoMatch, a decisão de engenharia mais importante foi a **fidelidade estrita à verdade**. A IA atua como uma lente de aumento para o que o profissional realmente fez, reorganizando e alinhando termos sem jamais inventar vivências. Ganhar uma entrevista com uma mentira gerada por IA só transfere a frustração para o momento da conversa técnica; nosso papel é dar segurança real ao candidato.

### A Força da Simplicidade (MVP sem Fricção)
Em um ecossistema digital saturado de formulários intermináveis, cadastros obrigatórios e muros de pagamento (*paywalls*), optamos deliberadamente pelo caminho oposto: **zero fricção e privacidade por padrão**. O candidato acessa, arrasta o arquivo, cola a vaga e obtém seu diagnóstico em segundos. O processamento do texto acontece no navegador, sem retenção desnecessária de documentos pessoais em servidores. Menos barreiras significam mais candidatos atendidos.

### Engenharia de Produto Orientada a Resultados
Mais do que apenas integrar um modelo de linguagem, o projeto demonstrou a importância de:
- **Saídas estruturadas (*Structured Outputs*):** Garantir que a IA retorne dados em esquemas JSON rigorosos, eliminando respostas quebradas e permitindo renderizar interfaces ricas e componentes nativos.
- **Geração de PDF determinística:** Separar a inteligência analítica da diagramação visual, permitindo que o PDF final em coluna única seja gerado no próprio cliente, no padrão exato que os leitores de ATS esperam.

---

"A tecnologia não deve substituir a história de um profissional; seu papel é garantir que essa história chegue limpa, clara e justa aos olhos de quem decide."



