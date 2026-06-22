# Auditoria Funcional do Protótipo Marcha — Área do Vendedor

## Objetivo

Este documento é um **briefing funcional** do protótipo Marcha (área do vendedor de uma plataforma de pagamentos/gateway). Ele mapeia, tela por tela, os **blocos, cards, botões, filtros, tabelas, ações e navegação** existentes para orientar o dev na implementação manual das telas.

O foco é **funcionalidade e utilidade para o vendedor** (vender mais, acompanhar caixa, agir rápido) — **não** estética, cor, fonte, borda ou layout. O protótipo serve para decidir **o que cada tela deve ter**, não para "ficar bonito" agora.

Lente da análise: **vendedor no dia a dia**, não administrador da plataforma.

---

## Achados Transversais

Padrões que se repetem em quase todas as telas:

### 1. Botões primários que hoje são apenas `toast()` (sem ação real)
Ações centrais que só exibem um aviso e não executam nada:
- `+ Novo Produto` (Produtos)
- `+ Adicionar Cliente` (Clientes)
- `Exportar` (Pedidos, Financeiro)
- `Gerar Relatório` (Pedidos)
- `Ver` detalhe de assinatura (Assinaturas)
- `Enviar defesa` / `Anexar arquivos` (Disputas / Infração)
- `Copiar link` de pagamento e de afiliado (Produtos)
- `Salvar` em Empresa, Perfil, Segurança
- `Alterar plano`, `Gerenciar cartões`, `Recarregar`, `Editar cartão` (Plano e Cobrança)

> **Regra para o dev:** todo botão acima deve ser **transformado em ação real** ou removido. Botão que não gera ação clara = remover ou transformar.

### 2. Dados mock inconsistentes
- **Marketplace**: stats incoerentes (Total 25 / Ativos 27 / Arquivados 2).
- **Cliente detalhe**: métricas zeradas (Gasto R$ 0,00, Pedidos 0) — mock não ligado.
- **Recompensas**: tudo zerado (Faturado R$ 0,00).
- **Partners**: KPIs e abas majoritariamente zerados/empty state.

### 3. Duplicações de conceito
- **Assinaturas** aparece 2x no menu (item real + dentro do grupo "Em breve").
- **Extrato** existe em Financeiro **e** dentro de Plano e Cobrança (Checkout/Cobrança).
- **Produtos**: abas internas (Meus/Shopify/Recorrentes/Afiliados) **duplicam** os subitens do menu.
- **Checkout**: nome usado para a tela de cobrança da plataforma colide com o futuro produto de checkout de vendas (ver seção dedicada).

### 4. Telas com excesso de filtros/cards
- **Pedidos**: 9 filtros sempre visíveis.
- **Dashboard**: 6 KPIs com mesmo peso (3 são conversão por método).
- **Clientes**: tabela com 9 colunas.
- **Integrações**: 3 níveis de abas aninhadas.
- **Partners**: 6 abas para um módulo de baixa frequência.

### 5. Telas que precisam virar ações reais
- **Disputas**: fluxo de defesa é o core e hoje é `toast()`.
- **Assinaturas**: detalhe e gestão (pausar/cancelar/reativar) não existem.
- **Produtos**: criação de produto e link de pagamento não funcionam.
- **Plano e Cobrança**: troca de plano / atualização de cartão não funcionam.

### 6. Conflito de conceito: Checkout vs Plano e Cobrança
O protótipo tem uma tela rotulada **"Checkout / Cobrança"** que, na prática, mostra **plano da plataforma, faturas, cartão e histórico de cobrança**. Isso é **Plano e Cobrança**, não Checkout.

- **O conteúdo atual pertence a "Plano e Cobrança"** (módulo de Conta).
- **O nome "Checkout" deve ser preservado** para o **produto de checkout de vendas** do vendedor (módulo futuro).

Detalhamento na seção da tela e em **Navegação Recomendada**.

---

## Auditoria por Tela

### Dashboard / Início

- **Função da tela:** dar ao vendedor, em segundos, o pulso do negócio — quanto vendeu, como converte e a tendência no tempo.
- **O que está bom:** KPIs certos (Volume de Vendas com líquido, Ticket Médio, Conversão Geral e por método Pix/Cartão/Boleto); filtro de período completo; botão ocultar valores; gráfico de evolução diária.
- **O que está sobrando:** 6 conversões com mesmo peso dos KPIs de receita (conversão por método é detalhe); botão "Participe da comunidade" no topo de decisão.
- **O que falta:** card **Top Produtos do período**; card **Recuperação de vendas** (Pix expirado/cartão recusado + recuperados); atalho para saldo/saque.
- **O que simplificar:** reduzir a **3 KPIs headline** (Volume, Ticket, Conversão Geral) e agrupar Pix/Cartão/Boleto num card "Conversão por método" expansível.
- **Navegação:** lugar certo, primeira do menu. Manter.
- **Prioridade:** **Ajustar levemente.**

### Financeiro

- **Função da tela:** controlar o caixa — saldo, saque, antecipação, extrato, taxas e chave Pix.
- **O que está bom:** 6 KPIs específicos de gateway (Disponível p/ saque com limite diário, Disponível p/ antecipação, Saldo bloqueado em disputa, Total reservado D+30, Total sacado, **Saúde da conta**); ações reais (`Solicitar Saque`, `Antecipar`, `Ver disputas`); subtabs claras; antecipação em lote.
- **O que está sobrando:** `Exportar` e `Limpar filtros` são `toast()`; subtab **Taxas** muito densa (tabela + prazos + cards por método).
- **O que falta:** **previsão da próxima liberação** (data + valor) no card de saque; **exportar/recibo** reais no Extrato; comprovante de saque.
- **O que simplificar:** manter visíveis 3 KPIs de caixa (Disponível, Antecipação, Saúde) e recolher Reservado/Total sacado em "Resumo da conta"; Taxas como consulta enxuta (1 tabela).
- **Navegação:** grupo Financeiro com 5 subitens está correto. Manter.
- **Prioridade:** **Ajustar levemente.**

### Pedidos

- **Função da tela:** encontrar, acompanhar e agir sobre transações.
- **O que está bom:** busca poderosa (E2E, UUID, Nome, CPF, Telefone); status cobre todos os estados reais; **detalhe do pedido excelente** (cliente, financeiro da transação, dados técnicos, carrinho, rastreio editável, andamento, tentativas, metadados).
- **O que está sobrando:** 9 filtros sempre visíveis (Parcelas, Origem, Rastreio, Tipo de produto são uso ocasional); `Exportar` é `toast()`; no detalhe, 2 botões de ajuda com mesma função.
- **O que falta:** **ações em massa** (reembolsar, exportar selecionados, marcar enviado); **rastreio** direto na linha.
- **O que simplificar:** filtros visíveis = **Buscar, Status, Pagamento, Período**; mover **Parcelas, Origem, Rastreio, Tipo de produto** para "Mais filtros". No detalhe, agrupar os 3 cards técnicos em "Detalhes técnicos" colapsável.
- **Navegação:** item de topo, correto. Manter.
- **Prioridade:** **Simplificar.**

### Produtos

- **Função da tela:** gerenciar o que é vendido e os links de pagamento.
- **O que está bom:** abas por natureza (Meus, Shopify, Recorrentes, Afiliados); colunas certas (ID, Produto, Link de pagamento, Valor, Status, Ações); empty state claro em Recorrentes.
- **O que está sobrando:** aba **"Produtos Shopify" abre como padrão** (deveria ser "Meus produtos"); `+ Novo Produto` e `Copiar link` são `toast()` — o botão mais importante não cria nada.
- **O que falta:** **métrica por produto** (vendas, faturamento, conversão); **copiar link de pagamento** funcional; **filtro por status** na lista.
- **O que simplificar:** tratar Shopify/Afiliados como **origem (filtro)**, não abas; reduzir 4 abas para 2 (Meus / Recorrentes) + filtro de origem.
- **Navegação:** o grupo Produtos no menu **duplica** as abas internas — escolher um só (abas **ou** subitens).
- **Prioridade:** **Reestruturar.**

### Clientes

- **Função da tela:** conhecer quem compra e quanto vale cada cliente.
- **O que está bom:** 3 stats úteis (Total clientes, Total pedidos, Total faturamento); detalhe com gasto, pedidos pagos, ticket médio e histórico.
- **O que está sobrando:** tabela com 9 colunas (CPF e Endereço inteiros poluem); `+ Adicionar Cliente` é `toast()` e, num gateway, cliente nasce da venda; detalhe com valores zerados.
- **O que falta:** filtros de **período, valor gasto (faixa), recorrência**; marcação de **melhores clientes (LTV)**.
- **O que simplificar:** colunas essenciais = **Nome, E-mail/Telefone, Nº Pedidos, Total Gasto, Ações**; mover CPF/Endereço ao detalhe; rever botão Adicionar.
- **Navegação:** item de topo, correto. Manter.
- **Prioridade:** **Simplificar.**

### Assinaturas

- **Função da tela:** acompanhar receita recorrente e saúde das assinaturas.
- **O que está bom:** KPIs ideais (Ativas, MRR, Canceladas/Churn %, Inadimplentes); colunas completas; filtro de status + busca.
- **O que está sobrando:** `Ver` (detalhe) é `toast()`; aparece **2x no menu**.
- **O que falta:** **detalhe real** (histórico de cobranças, próxima cobrança, pausar/cancelar/reativar, atualizar cartão); **recuperação de inadimplência (dunning)**.
- **O que simplificar:** já está enxuta — ligar o detalhe e remover a duplicata do menu.
- **Navegação:** manter item de topo; **remover** entrada duplicada em "Em breve".
- **Prioridade:** **Ajustar levemente.**

### Disputas

- **Função da tela:** defender chargebacks/MEDs antes do prazo e reduzir perda.
- **O que está bom:** stats certos (Em aberto, Valor em disputa, Aguardando defesa); coluna **Data Limite** + ordenação por prazo; detalhe (Infração) completo com fluxo de defesa e anexo.
- **O que está sobrando:** marcada **"Em construção"** no menu apesar de já rica; defesa (`Enviar defesa`, `Anexar`) é `toast()`.
- **O que falta:** **contador de prazo** (faltam X dias) na lista; **alerta** de prazo próximo no Dashboard.
- **O que simplificar:** filtros ok; `Ordenação` pode ir para "Mais filtros".
- **Navegação:** em "Gestão" faz sentido; tirar o selo "Em construção" quando a defesa for real.
- **Prioridade:** **Ajustar levemente.**

### Marketplace

- **Função da tela:** descobrir produtos para afiliar/revender.
- **O que está bom:** abas (Explorar / Afiliações / Arquivados) e filtros de catálogo.
- **O que está sobrando:** **stats inconsistentes** (Total 25 / Ativos 27); para o vendedor focado na própria operação, é secundário e não deve competir com Pedidos/Financeiro.
- **O que falta:** se mantido, **detalhe do produto/afiliação** com comissão e regras.
- **O que simplificar:** reduzir filtros a **Buscar, Tipo, Status**; afiliação só na aba Afiliações.
- **Navegação:** manter em "Gestão", abaixo da operação. Não promover.
- **Prioridade:** **Simplificar.**

### Partners (Indique)

- **Função da tela:** programa de indicação/afiliados da plataforma (ganhar comissão indicando novos vendedores).
- **O que está bom:** calculadora de projeção, link de indicação com copiar, níveis/prêmios, abas organizadas.
- **O que está sobrando:** **6 abas** para algo majoritariamente zerado; marcada "Em construção" (quase tudo empty/`toast()`); calculadora com promessa de marketing ("R$ 1,35 mi/ano").
- **O que falta:** nada essencial à operação de vender — é retenção/growth da plataforma.
- **O que simplificar:** colapsar para **1 tela** (link + status); esconder a calculadora atrás de "Simular ganhos".
- **Navegação:** mais valor para a plataforma do que para o vendedor diário — manter em "Gestão", baixa prioridade.
- **Prioridade:** **Simplificar.**

### Recompensas

- **Função da tela:** gamificação — metas de faturamento com prêmios (10K → 1 mi).
- **O que está bom:** jornada clara com etapas e barra de progresso; motivacional.
- **O que está sobrando:** tudo zerado; acessada por widget no topo (fora do menu), o que a "esconde"; card "Comece sua jornada" sem ação.
- **O que falta:** ligar ao faturamento real e dar **próxima ação** ("faltam R$ X para a pulseira").
- **O que simplificar:** é um **card**, não uma tela — integrar ao Dashboard ou ao Perfil.
- **Navegação:** não justifica item próprio; decidir um único lugar (bloco no Dashboard ou widget).
- **Prioridade:** **Reestruturar.**

### Checkout (produto de vendas) — RESERVADO

- **Função da tela:** **módulo futuro de checkout de vendas do vendedor.** Deve abrigar **criação/configuração de checkout, links de pagamento, métodos de pagamento, ofertas, order bump, upsell, recuperação e configurações de conversão.**
- **Status atual no protótipo:** **não implementado como produto de vendas.** O rótulo "Checkout" hoje está sendo usado por uma tela de cobrança da plataforma (ver "Plano e Cobrança" abaixo) — isso é um **conflito de conceito**.
- **O que está bom:** —  (a definir; módulo a construir).
- **O que está sobrando:** o **nome** está emprestado para a tela errada.
- **O que falta:** o módulo inteiro de checkout de vendas (configuração de checkout, links, ofertas, bump, upsell, recuperação, conversão).
- **O que simplificar:** —  (não construir antes de separar do módulo de cobrança).
- **Navegação:** pertence à **Operação / produto de vendas** (não à Conta). Preservar o nome "Checkout" para este módulo.
- **Prioridade:** **Reestruturar** (definir e construir como produto de vendas; primeiro, liberar o nome separando da cobrança).

### Plano e Cobrança (atual tela "Checkout / Cobrança")

- **Função da tela:** gerir o **plano e a cobrança do vendedor pela plataforma** — mensalidade, taxa por pedido, cartão em arquivo, faturas e histórico.
- **O que está bom:** plano, taxa por pedido, cartão em arquivo, próxima fatura, histórico com stats (Pendentes/Em atraso/Pagas/Total).
- **O que está sobrando:** abas **Extrato** (colide com Financeiro→Extrato) e **Automação** (sem função clara); botões majoritariamente `toast()`; **o rótulo "Checkout" é conflito de conceito** — este conteúdo é Plano e Cobrança.
- **O que falta:** ações reais de **trocar plano** e **atualizar cartão**.
- **O que simplificar:** reduzir a **Faturas + Histórico**; remover Extrato (duplicado) e Automação (ou definir função).
- **Navegação:** é **Conta**, não operação. **Mover para o grupo "Conta"** como **"Plano e Cobrança"**. **Não** usar o nome "Checkout" aqui — ele fica reservado para o produto de vendas.
- **Prioridade:** **Reestruturar** (renomear para Plano e Cobrança, reagrupar em Conta, enxugar abas, ações reais).

### Empresa

- **Função da tela:** dados cadastrais, representante legal, documentos e compliance (KYC).
- **O que está bom:** campos completos, documentos com upload, status de compliance.
- **O que está sobrando:** `Salvar` é `toast()`; tela de baixa frequência ocupando destaque de navegação.
- **O que falta:** **status de verificação por documento** (aprovado/pendente/recusado), ligado ao KYC.
- **O que simplificar:** ok como formulário — agrupar com Perfil/Segurança.
- **Navegação:** mover para grupo **"Conta"** (já chega via `go('config')`, confirmando que é settings).
- **Prioridade:** **Ajustar levemente.**

### Perfil

- **Função da tela:** dados pessoais do usuário e preferências de notificação.
- **O que está bom:** dados pessoais, endereço, status da conta/documentação, toggles de notificação (e-mail/WhatsApp).
- **O que está sobrando:** `Salvar` (Perfil e Preferências) são `toast()`; sobreposição com Empresa (endereço/contato repetidos).
- **O que falta:** trocar e-mail/telefone com verificação.
- **O que simplificar:** unir **Perfil + Empresa + Segurança** numa área "Conta" com abas (dados se repetem hoje em 3 telas).
- **Navegação:** grupo "Conta".
- **Prioridade:** **Ajustar levemente.**

### Segurança

- **Função da tela:** 2FA, troca de senha e sessões ativas.
- **O que está bom:** 2FA, troca de senha, **sessões ativas com encerrar** — exatamente o necessário.
- **O que está sobrando:** `Salvar`/`Encerrar sessão` são `toast()`.
- **O que falta:** **log de segurança** (logins, mudanças sensíveis) — aqui entra a tela **Auditoria** (hoje separada e com cara de admin), que faz mais sentido como aba "Atividade".
- **O que simplificar:** virar **aba** dentro de "Conta", não tela isolada.
- **Navegação:** grupo "Conta"; considerar **fundir Auditoria** como "Atividade da conta".
- **Prioridade:** **Ajustar levemente.**

### Integrações / API / Webhooks

- **Função da tela:** conectar a Marcha a sistemas externos — webhooks, chaves de API, integrações.
- **O que está bom:** webhooks com **catálogo de eventos bem definido** (ORDER.*, PAYMENT.*, WITHDRAWAL.*); chaves de API com ambiente (Produção/Sandbox), status e **IP whitelist** para saque/estorno; sub-áreas API (QR/BaaS/Conta) e App Store.
- **O que está sobrando:** **3 camadas de abas** (Webhook/API/AppStore → QR/BaaS/Conta) — navegação profunda demais; **App Store vazia**; Documentação como `toast()`.
- **O que falta:** **teste de webhook** e **log de entregas** (último 2xx/erro); status de saúde da integração.
- **O que simplificar:** achatar para **2 níveis** (API | Webhooks | Integrações); BaaS/Conta como seções dentro de "API".
- **Navegação:** em "Desenvolvedores" está certo; reduzir profundidade.
- **Prioridade:** **Simplificar.**

---

## Mapa de Enxugamento (Top 10 simplificações)

1. **Pedidos:** 9 filtros → 4 visíveis (Buscar, Status, Pagamento, Período); resto em "Mais filtros".
2. **Dashboard:** 6 KPIs → 3 headline + 1 card "Conversão por método" agrupado.
3. **Produtos:** default volta para "Meus produtos"; decidir entre abas **ou** subitens de menu (hoje duplicado).
4. **Partners:** 6 abas → 1 tela (link + status), simulador opcional.
5. **Recompensas:** virar **bloco**, não tela.
6. **Plano e Cobrança:** 4 abas → 2 (Faturas, Histórico); remover Extrato duplicado e Automação.
7. **Conta:** fundir **Perfil + Empresa + Segurança (+ Auditoria)** numa área única com abas.
8. **Clientes:** 9 colunas → 5 essenciais; CPF/Endereço só no detalhe.
9. **Integrações:** 3 níveis de abas → 2.
10. **Menu:** remover duplicatas (Assinaturas 2x; "Checkout" como cobrança); desfazer o grupo confuso "Em breve".

---

## Mapa de Acréscimos (só o que é realmente útil)

1. **Dashboard:** card **Top Produtos** + card **Recuperação de vendas**.
2. **Financeiro:** **previsão da próxima liberação** (data+valor) no card de saque; **Exportar/Comprovante** reais.
3. **Produtos:** **métrica por produto** (vendas/faturamento) + **copiar link de pagamento** funcional.
4. **Assinaturas:** **detalhe real** (histórico de cobranças, pausar/cancelar/reativar) + **dunning** para inadimplentes.
5. **Disputas:** **contador de prazo** na lista + **alerta de prazo** no Dashboard.
6. **Pedidos:** **ações em massa** (reembolsar/exportar/marcar enviado).
7. **Clientes:** filtros de **período/valor/recorrência** + marcação de melhores clientes (LTV).
8. **Integrações:** **teste e log de entrega de webhook**.
9. **Checkout (produto):** módulo de checkout de vendas (configuração, links, ofertas, order bump, upsell, recuperação, conversão).

> Transformar botões `toast()` em ações reais é **correção**, não recurso novo — mas é pré-requisito para quase tudo acima.

---

## Navegação Recomendada

### Operação
- Início (Dashboard)
- Pedidos
- Assinaturas
- Clientes
- Produtos *(abas internas: Meus · Recorrentes · Afiliados; Shopify = filtro de origem)*
- **Checkout** *(produto de vendas — módulo futuro: criação/configuração de checkout, links de pagamento, métodos, ofertas, order bump, upsell, recuperação, conversão)*

### Financeiro
- Saldo & Saques
- Antecipação
- Extrato
- Taxas
- Chave Pix

### Gestão
- Disputas
- Recuperação de vendas
- Marketplace
- Partners / Indique *(baixa prioridade)*

### Desenvolvedores
- Central de Apps / Integrações
- API
- Webhooks

### Conta *(menu do avatar, fora da barra lateral)*
- Perfil
- Empresa
- Segurança & Atividade *(absorve Auditoria)*
- **Plano e Cobrança** *(mensalidade da plataforma, faturas, cartão, histórico, plano contratado — é o conteúdo atual da tela "Checkout/Cobrança")*

**Princípios:** operação no topo; caixa em segundo; gestão/risco depois; dev separado; **cadastro/conta sai da barra lateral** para o menu da conta. Remove duplicatas e o grupo confuso "Em breve".

**Distinção crítica:**
- **Checkout** = produto de vendas do vendedor → fica em **Operação**.
- **Plano e Cobrança** = cobrança da plataforma sobre o vendedor → fica em **Conta**.
- São módulos **diferentes**; o nome "Checkout" **não** deve ser reaproveitado para a cobrança.

---

## Ordem de Implementação

1. **Pedidos** — lista + detalhe + ações reais; coração transacional.
2. **Financeiro** — saldo, saque, antecipação, extrato; caixa.
3. **Dashboard** — KPIs enxutos + Top Produtos + Recuperação (depende de Pedidos/Financeiro).
4. **Produtos** — CRUD real, link de pagamento, métrica por produto.
5. **Clientes** — lista + detalhe ligado a Pedidos.
6. **Assinaturas** — detalhe + dunning.
7. **Disputas** — defesa real + prazos.
8. **Integrações / API / Webhooks** — habilita quem vende por API/externo.
9. **Conta** — Perfil/Empresa/Segurança/**Plano e Cobrança** consolidados.
10. **Marketplace / Partners / Recompensas** — growth/secundário, por último.

> O módulo **Checkout (produto de vendas)** entra como trilha própria quando for priorizado; antes disso, o pré-requisito é **separar o nome "Checkout" da tela de cobrança** (que vira "Plano e Cobrança" no passo 9).

**Lógica:** primeiro o que **registra dinheiro** (Pedidos → Financeiro → Dashboard), depois o que **gera venda** (Produtos → Clientes → Assinaturas), depois **protege** (Disputas), depois **conecta** (Integrações), e por fim **conta e extras**.

---

## Observação Final

Esta auditoria **não é ordem para cortar módulos**. Nenhum módulo é descartado — Marketplace, Partners e Recompensas continuam válidos, apenas com prioridade e destaque menores.

O objetivo é **organizar prioridade, clareza e implementação futura**: enxugar o que polui, ligar o que hoje é `toast()`, separar conceitos sobrepostos (com destaque para **Checkout** vs **Plano e Cobrança**) e dar ao dev uma ordem racional de construção. As decisões finais de escopo permanecem com o time de produto.
