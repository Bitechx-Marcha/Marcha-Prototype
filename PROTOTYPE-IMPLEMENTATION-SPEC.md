# Especificação Funcional de Implementação — Prototype Marcha

> Documento irmão de `PROTOTYPE-PRODUCT-AUDIT.md`. A auditoria diz **o que cada tela deve ter e por quê**; esta spec diz **o que o dev precisa construir** para transformar o protótipo (`index.html`, HTML único com dados mock) em produto real.

---

## Como usar este documento

Este é o **checklist de implementação** para o time de engenharia. Cada tela tem um bloco padronizado com: objetivo, prioridade, blocos/cards, filtros, tabelas/colunas, botões/ações, modais, estados vazios, estados de erro, dados de backend, permissões e observações de produto.

Regras de leitura:
- O **`index.html` é referência visual/funcional**, não fonte da verdade de regras de negócio. Onde a spec e o protótipo divergirem, **vale a spec**.
- Tudo marcado no protótipo como `toast('…')` é **placeholder** — virar ação real (lista consolidada na seção própria).
- Dados são **mockados em JS** no protótipo — virar entidades de backend (lista consolidada na seção própria).
- "MVP" significa: necessário para o vendedor operar e receber dinheiro com segurança. O resto é incremento.

---

## Priorização Geral

### MVP (operar + receber + proteger)
- Pedidos
- Detalhe do Pedido
- Financeiro (Saldo/Saque/Antecipação/Extrato/Taxas/Chave Pix)
- Dashboard
- Produtos
- Clientes
- Detalhe do Cliente
- Conta / Perfil
- Empresa / KYC
- Segurança / Atividade
- Plano e Cobrança

### Fase 2 (receita recorrente + risco + integração)
- Assinaturas
- Disputas
- Detalhe da Disputa / MED
- Integrações / API / Webhooks

### Fase 3 (produto de vendas)
- Checkout de Vendas
- Recuperação de Vendas

### Futuro / Growth
- Marketplace
- Partners
- Recompensas

---

## Especificação por Tela

### Dashboard

- **Objetivo:** pulso do negócio em segundos — receita, conversão, tendência, de onde vem o dinheiro e o que dá para recuperar.
- **Prioridade:** MVP (depois de Pedidos e Financeiro existirem).
- **Blocos/cards necessários:**
  - 3 KPIs headline: Volume de Vendas (bruto + líquido), Ticket Médio, Conversão Geral.
  - 1 card "Conversão por método" (Pix/Cartão/Boleto) expansível.
  - Gráfico de evolução diária (vendas + faturamento).
  - Card **Top Produtos** do período (top 5: nome, vendas, faturamento, % do total, tendência).
  - Card **Recuperação de vendas** (Pix expirado, boleto não pago, cartão recusado, recuperados — qtd + R$).
- **Filtros necessários:** período (Hoje, Ontem, 7/14/30 dias, Esse mês, Mês passado, Todos, Personalizada).
- **Tabela/colunas:** Top Produtos → #, Produto, Vendas, Faturamento, % do total, Tendência.
- **Botões/ações:** ocultar valores (toggle blur); "Ver produtos" → Produtos; "Recuperar vendas" → Recuperação de Vendas; atalho "Sacar" → Financeiro.
- **Modais:** nenhum próprio (reaproveita saque do Financeiro).
- **Estados vazios:** sem vendas no período → KPIs zerados + "Nenhuma venda no período" nos cards.
- **Estados de erro:** falha ao carregar métricas → card com "Não foi possível carregar" + retry.
- **Dados do backend:** agregados por período (volume, líquido, ticket, conversão geral e por método), série diária, ranking de produtos por faturamento, contadores de recuperação.
- **Permissões:** somente dados da **loja ativa** do vendedor; respeitar troca de loja.
- **Observações de produto:** mover "Participe da comunidade" para fora do topo de decisão. Dashboard é leitura — toda ação leva a outra tela.

---

### Pedidos

- **Objetivo:** encontrar, acompanhar e agir sobre transações.
- **Prioridade:** MVP (tela 1).
- **Blocos/cards:** barra de filtros, chips de filtros ativos, tabela paginada, rodapé (itens por página + paginação).
- **Filtros necessários:**
  - Visíveis: Buscar (E2E, UUID, Nome, CPF, Telefone), Status, Tipo de pagamento, Período.
  - Em "Mais filtros": Tipo de produto, Rastreio, Origem, Parcelas, Data (range).
  - Status: Processando, Pendente, Pago, Recusado, Reembolso, Cancelado, Expirado, MED, Disputa, Chargeback.
- **Tabela/colunas:** ID/UUID, Cliente, Produto, Status, Pagamento, Valor, Data, Ações. (Rastreio como coluna opcional para vendedor de físico.)
- **Botões/ações:** Buscar/filtrar; Limpar filtros (real); **Exportar** (CSV/XLSX real); **Gerar Relatório** (seleção de colunas → arquivo real); abrir detalhe; **ações em massa** (reembolsar, marcar enviado, exportar selecionados).
- **Modais:** Gerar Relatório (seleção de colunas — já existe `REPORT_COLS`); confirmação de reembolso em massa.
- **Estados vazios:** nenhum pedido / nenhum resultado de filtro → mensagem + limpar filtros.
- **Estados de erro:** falha de carga → retry; exportação falha → toast de erro real.
- **Dados do backend:** lista de transações com todos os campos de filtro/coluna; paginação server-side; contagem total; geração de relatório assíncrona.
- **Permissões:** apenas pedidos da loja ativa; reembolso exige permissão financeira.
- **Observações de produto:** busca é o recurso mais usado — manter no topo. Reembolso é irreversível → sempre confirmar.

---

### Detalhe do Pedido

- **Objetivo:** visão 360º de uma transação para suporte, conferência e ação.
- **Prioridade:** MVP.
- **Blocos/cards:** Informações do cliente; Forma de pagamento; Resumo do pedido (subtotal, frete, descontos, total, deduções, líquido); Financeiro da transação (bruto, taxa, adquirência, comissão, lucro); Dados técnicos (adquirente, IDs, referência externa); Status & datas; Carrinho (itens); Rastrear pacote (editável); Precisa de ajuda; Andamento do pedido (steps); Tentativas de processamento; Metadados.
- **Filtros:** N/A.
- **Tabela/colunas:** itens do carrinho (Quantidade, Preço unit., Total, Comissão, Taxa).
- **Botões/ações:** Voltar; Ver item; **Alterar rastreio** (salvar real); Falar com suporte (real); reembolsar/estornar (real, condicional ao status); ações contextuais por status (`detActions`).
- **Modais:** confirmação de reembolso; edição de rastreio (inline já existe).
- **Estados vazios:** sem rastreio (vendedor adiciona); sem tentativas/metadados → ocultar seção.
- **Estados de erro:** salvar rastreio falha → manter edição + erro; reembolso falha → erro claro.
- **Dados do backend:** transação completa, itens, tentativas por adquirente, timeline de status, metadados de criação, dados de rastreio.
- **Permissões:** apenas pedido da loja ativa (atenção a **IDOR** por ID de pedido); reembolso exige permissão.
- **Observações de produto:** agrupar os 3 cards técnicos em "Detalhes técnicos" colapsável; unificar os 2 botões de ajuda em 1.

---

### Financeiro

- **Objetivo:** controlar o caixa — saldo, saque, antecipação, extrato, taxas, chave Pix.
- **Prioridade:** MVP (tela 2).
- **Blocos/cards (KPIs):** Disponível para saque (com limite diário/noturno), Disponível para antecipação, Saldo bloqueado em disputa, Total reservado (D+30), Total sacado, Saúde da conta (chargeback/MED/estorno). **Adicionar:** previsão da próxima liberação (data + valor) no card de saque.
- **Subtabs:** Saques, Antecipação, Extrato, Taxas, Chave Pix.
- **Filtros necessários:** Buscar (ID, E2E, Chave Pix); Tipo de Chave (CPF/CNPJ/Telefone/E-mail/Aleatória); Status (Aprovado/Negado/Processando/Pendente/Pago/Falha/Cancelado); Período; Data (range). Extrato: Tipo de movimento (Entrada/Saída).
- **Tabela/colunas:**
  - Saques: ID, E2E, Tipo de Chave, Chave, Valor, Status, Data solicitação, Data pagamento, Ações.
  - Antecipação (disponível): Cliente, Valor, Cobrança, Criação, Ação (+ seleção em lote).
  - Antecipação (histórico): Status, Valor líquido, Cobrança, Previsão, Solicitação.
  - Extrato: Transação, Tipo, Valor, Registro (+ cards Saldo inicial/Entradas/Saídas/Saldo final).
- **Botões/ações:** Solicitar Saque (modal real); Antecipar / Antecipar tudo / Antecipar selecionados (real); Ver disputas (→ Disputas); Criar chave Pix (modal real); Solicitar alteração de taxas (real); **Exportar** (real); comprovante de saque (download).
- **Modais:** Saque, Antecipação, Criar Chave Pix, Taxas de parcelamento (`openParc`).
- **Estados vazios:** sem saques/antecipações/movimentos/chaves → empty state por subtab.
- **Estados de erro:** saldo insuficiente; saque negado; limite diário excedido; chave Pix inválida.
- **Dados do backend:** saldos (disponível, a liberar, reservado, bloqueado), limites diários/noturnos, recebíveis antecipáveis, extrato de movimentos, tabela de taxas e prazos de liquidação, chaves Pix do recebedor, saúde da conta (índices).
- **Permissões:** saque/antecipação exigem **2FA** e respeitam **IP whitelist**; KYC aprovado obrigatório; permissão financeira por membro.
- **Observações de produto:** caixa é a tela mais sensível — toda ação financeira passa por confirmação + autenticação. Saúde da conta deve alimentar alertas no Dashboard.

---

### Produtos

- **Objetivo:** gerenciar o que é vendido e os links de pagamento.
- **Prioridade:** MVP (tela 4).
- **Blocos/cards:** abas por origem/natureza; tabela; paginação; empty states.
- **Filtros necessários:** Buscar (nome); **Status** (Ativo/Desativado/Sem estoque); origem (Meus/Shopify/Afiliados como **filtro**, não abas separadas); Tipo (Físico/Digital/Serviço).
- **Tabela/colunas:** ID, Produto, Link de pagamento, Valor, Status, Ações. Afiliados: + Produtor, Comissão.
- **Botões/ações:** **+ Novo Produto** (CRUD real — criar/editar/ativar/desativar); **Copiar link de pagamento** (real); copiar link de afiliado (real).
- **Modais:** criação/edição de produto (nome, preço, tipo, estoque, link, oferta).
- **Estados vazios:** "Nenhum produto recorrente" (já existe) e equivalentes por aba.
- **Estados de erro:** falha ao salvar produto; preço inválido; SKU duplicado.
- **Dados do backend:** catálogo de produtos da loja, preços, status, links de pagamento, estoque, vínculos de afiliação e comissão; agregados de venda por produto (para métrica).
- **Permissões:** apenas produtos da loja ativa; afiliados veem comissão, não custo.
- **Observações de produto:** **default = "Meus produtos"** (hoje abre em Shopify). Adicionar **métrica por produto** (vendas/faturamento) ao menos no detalhe — é onde se decide escalar/pausar.

---

### Clientes

- **Objetivo:** conhecer quem compra e o valor de cada cliente.
- **Prioridade:** MVP (tela 5).
- **Blocos/cards:** 3 stats (Total de clientes, Total de pedidos, Total faturamento); tabela; paginação.
- **Filtros necessários:** Buscar (nome, ID, CPF); **Período**; **Faixa de valor gasto**; **Recorrência** (1ª compra vs recorrente).
- **Tabela/colunas (enxuta):** Nome, E-mail/Telefone, Nº Pedidos, Total Gasto, Ações. (CPF e Endereço → detalhe.)
- **Botões/ações:** abrir detalhe; rever `+ Adicionar Cliente` (num gateway o cliente nasce da venda — manter só se houver caso real); exportar.
- **Modais:** (se mantido) adicionar/editar cliente.
- **Estados vazios:** sem clientes / sem resultado.
- **Estados de erro:** falha de carga; export falha.
- **Dados do backend:** clientes derivados das transações, agregados (gasto, nº pedidos, ticket, recência), dados de contato.
- **Permissões:** apenas clientes da loja ativa; LGPD — dados pessoais protegidos por permissão.
- **Observações de produto:** marcar **melhores clientes (LTV)** para remarketing.

---

### Detalhe do Cliente

- **Objetivo:** ficha do cliente com histórico e valor.
- **Prioridade:** MVP.
- **Blocos/cards:** ficha (avatar, nome, e-mail, CPF/CNPJ, telefone); métricas (Gasto, Pedidos pagos, Ticket médio); histórico de transações.
- **Filtros:** (opcional) por status/período no histórico.
- **Tabela/colunas:** Valor, Produto, Status.
- **Botões/ações:** Voltar; abrir pedido a partir da linha; (opcional) contato/remarketing.
- **Modais:** N/A.
- **Estados vazios:** cliente sem pedidos pagos → métricas zeradas com texto claro (hoje aparece zerado por mock não ligado — **ligar ao backend**).
- **Estados de erro:** falha de carga.
- **Dados do backend:** cliente + agregados + lista de transações do cliente.
- **Permissões:** IDOR por ID de cliente; apenas loja ativa.
- **Observações de produto:** as métricas precisam vir reais (hoje são placeholder).

---

### Assinaturas

- **Objetivo:** acompanhar e gerir receita recorrente.
- **Prioridade:** Fase 2.
- **Blocos/cards:** KPIs (Ativas, MRR, Canceladas/Churn %, Inadimplentes); tabela; busca/filtro.
- **Filtros necessários:** Buscar (cliente, plano, ID); Status (Todas/Ativa/Pendente/Cancelada).
- **Tabela/colunas:** Cliente, Valor, Método, Status, Criação, ID assinatura, Produto, Ações.
- **Botões/ações:** **Ver detalhe** (real); pausar, cancelar, reativar, atualizar cartão; **dunning** (cobrar inadimplentes).
- **Modais:** detalhe da assinatura (histórico de cobranças, próxima cobrança, ações); confirmação de cancelamento.
- **Estados vazios:** sem assinaturas; sem inadimplentes.
- **Estados de erro:** falha ao pausar/cancelar; cobrança recusada.
- **Dados do backend:** assinaturas, ciclos de cobrança, MRR/churn calculados, status de inadimplência, tentativas de cobrança.
- **Permissões:** apenas loja ativa; ação de cancelar exige permissão.
- **Observações de produto:** remover duplicata do menu; dunning é alavanca direta de receita.

---

### Disputas

- **Objetivo:** defender chargebacks/MEDs antes do prazo e reduzir perda.
- **Prioridade:** Fase 2.
- **Blocos/cards:** stats (Em aberto, Valor em disputa, Aguardando defesa); tabela; tabs Situação (Aberta/Fechada).
- **Filtros necessários:** Buscar (documento, EndToEnd); Tipo (MED/Chargeback); Status (Pendente/Defendida/Aguardando instituição); Data (range); Ordenação (por prazo — pode ir a "Mais filtros").
- **Tabela/colunas:** Status, Data de Criação, **Data Limite** (+ contador "faltam X dias"), Valor, Documento Pagador, EndToEnd, Tipo da Infração, Ações.
- **Botões/ações:** abrir detalhe; ordenar por prazo.
- **Modais:** N/A (ação no detalhe).
- **Estados vazios:** nenhuma disputa em aberto.
- **Estados de erro:** falha de carga.
- **Dados do backend:** disputas/infrações com prazos, valores, documentos, tipo, status da instituição.
- **Permissões:** apenas loja ativa; valor fica bloqueado no Financeiro até resolução.
- **Observações de produto:** prazo é tudo — expor contador na lista e **alerta no Dashboard**; remover selo "Em construção" quando a defesa for real.

---

### Detalhe da Disputa / MED

- **Objetivo:** apresentar defesa com anexos dentro do prazo.
- **Prioridade:** Fase 2.
- **Blocos/cards:** status + posição da instituição; dados (datas, documentos pagador/recebedor, resumo, detalhes da abertura); transação relacionada (pagador, valor, CPF, data, EndToEnd, TXID); webhooks disparados (colapsável); **defesa** (textarea + anexos); histórico de defesa.
- **Filtros:** N/A.
- **Tabela/colunas:** N/A (campos chave-valor).
- **Botões/ações:** Voltar; Histórico de transações; Infrações desse documento; Atualizar; **Anexar arquivos** (real); **Enviar defesa** (real).
- **Modais:** confirmação de envio de defesa.
- **Estados vazios:** "Nenhum anexo", "Nenhuma defesa enviada", "Nenhum webhook disparado".
- **Estados de erro:** prazo expirado (bloquear envio); upload falha; defesa rejeitada.
- **Dados do backend:** infração completa, transação, anexos, log de webhooks da disputa, histórico de defesas.
- **Permissões:** IDOR por ID de infração; apenas loja ativa.
- **Observações de produto:** o core (enviar defesa/anexar) é placeholder hoje — é a função que justifica a tela.

---

### Checkout de Vendas

- **Objetivo:** **produto de checkout de vendas do vendedor** (não confundir com Plano e Cobrança). Criar e configurar o checkout que o cliente final usa.
- **Prioridade:** Fase 3.
- **Blocos/cards:** criação/configuração de checkout; **links de pagamento**; **métodos de pagamento** (Pix/Cartão/Boleto + parcelas); **ofertas**; **order bump**; **upsell/downsell**; **recuperação** (vínculo com Recuperação de Vendas); **configurações de conversão** (campos do form, prova social, timer, redirecionos).
- **Filtros necessários:** lista de checkouts por status/produto.
- **Tabela/colunas:** Checkout, Produto, Conversão, Status, Ações.
- **Botões/ações:** criar/editar checkout; gerar link de pagamento; ativar/desativar; pré-visualizar; configurar bump/upsell.
- **Modais:** editor de checkout; configuração de oferta; configuração de bump/upsell.
- **Estados vazios:** nenhum checkout criado.
- **Estados de erro:** configuração inválida; método de pagamento não habilitado (depende de KYC/contrato).
- **Dados do backend:** checkouts, ofertas, bumps, upsells, links, métricas de conversão por checkout.
- **Permissões:** apenas loja ativa; métodos dependem de KYC e taxas contratadas.
- **Observações de produto:** **o nome "Checkout" é reservado para este módulo.** Hoje não existe como produto de vendas — só há uma tela de cobrança mal rotulada (ver Plano e Cobrança). Construir em Operação, não em Conta.

---

### Recuperação de Vendas

- **Objetivo:** recuperar vendas não concluídas (Pix abandonado, cartão recusado, chargeback) via mensagens/links.
- **Prioridade:** Fase 3.
- **Blocos/cards:** serviços de recuperação (cards com toggle on/off):
  - Recuperação de Pix via WhatsApp (preço por mensagem).
  - Recuperação automática de Chargebacks (preço por tentativa).
  - Recuperação de carrinho abandonado com IA (em breve).
  - Recuperação de vendas negadas pelo banco com IA (em breve).
  - Compra Protegida (grátis).
  - Seletor de loja.
- **Filtros:** por loja.
- **Tabela/colunas:** (futuro) histórico de recuperações: tipo, qtd, recuperado R$, taxa de sucesso.
- **Botões/ações:** ativar/desativar serviço (real); Salvar configuração (real); Saiba mais.
- **Modais:** confirmação de ativação (com custo).
- **Estados vazios:** serviços "em breve" sem toggle.
- **Estados de erro:** falha ao salvar; saldo insuficiente para serviço pago.
- **Dados do backend:** configuração de serviços por loja, eventos de recuperação, custos, resultados.
- **Permissões:** apenas loja ativa; serviços pagos debitam saldo/cobrança.
- **Observações de produto:** alimenta o card "Recuperação" do Dashboard. Integra com Checkout de Vendas.

---

### Integrações / API / Webhooks

- **Objetivo:** conectar a Marcha a sistemas externos com segurança.
- **Prioridade:** Fase 2.
- **Blocos/cards:** abas **Webhooks | API | Integrações (App Store)** (achatar — máx. 2 níveis).
  - Webhooks: tabela + criação com catálogo de eventos.
  - API: chaves (ambiente, status, prefixo, último uso), IP whitelist, credenciais BaaS/Conta como **seções**.
  - App Store: grid de integrações + busca.
- **Filtros necessários:** busca de integrações; (opcional) filtro de eventos.
- **Tabela/colunas:**
  - Webhooks: Nome, Ambiente, Prefixo, Status, Data de Criação, Último uso, Ações.
  - API keys: Nome/Descrição, Ambiente, Prefixo/Credencial, Status, IPs permitidos, Último uso, Ações.
- **Botões/ações:** Criar webhook (modal real, eventos ORDER.*/PAYMENT.*/WITHDRAWAL.*); Criar chave de API (real); Regenerar credenciais (real); Adicionar/remover IP (real); Editar permissões da credencial (real); **Testar webhook** + **ver log de entregas** (novo).
- **Modais:** Criar webhook (`whModal`), Criar chave (`apiModal`), Permissões da credencial (`permModal`).
- **Estados vazios:** App Store vazia (popular); nenhum webhook/chave.
- **Estados de erro:** endpoint inválido; entrega de webhook falhando (mostrar último erro); IP inválido.
- **Dados do backend:** webhooks + assinaturas de eventos + log de entregas; chaves de API com escopo/ambiente/IP; credenciais BaaS; catálogo de integrações.
- **Permissões:** **crítico** — criação/rotação de chave e IP whitelist exigem 2FA; segredos exibidos uma única vez; escopo por permissão.
- **Observações de produto:** reduzir aninhamento de abas; **log de entregas de webhook** é o que falta para depurar.

---

### Marketplace

- **Objetivo:** descobrir produtos para afiliar/revender.
- **Prioridade:** Futuro/Growth.
- **Blocos/cards:** abas (Explorar/Afiliações/Arquivados); stats; grid de produtos.
- **Filtros necessários (enxuto):** Buscar, Tipo, Status. (Tipo de afiliação só na aba Afiliações.)
- **Tabela/colunas:** grid de cards (produto, produtor, comissão, preço).
- **Botões/ações:** afiliar-se; solicitar afiliação; arquivar; ver detalhe.
- **Modais:** detalhe do produto/afiliação (comissão, regras).
- **Estados vazios:** nenhum produto/afiliação.
- **Estados de erro:** falha ao afiliar.
- **Dados do backend:** catálogo público, regras de afiliação, comissão, status.
- **Permissões:** apenas vendedor autenticado; comissões corretas por afiliado.
- **Observações de produto:** **corrigir stats inconsistentes** (Total 25/Ativos 27); manter abaixo da operação.

---

### Partners

- **Objetivo:** programa de indicação/afiliados da plataforma.
- **Prioridade:** Futuro/Growth.
- **Blocos/cards (enxuto):** 1 tela — link de indicação + status (indicados, comissões, saldo, saques); simulador opcional atrás de "Simular ganhos".
- **Filtros:** N/A.
- **Tabela/colunas:** Indicados (Nome, E-mail, Taxas, Total em comissões, Data de chegada).
- **Botões/ações:** copiar link (real); indicar usuário; solicitar saque (real).
- **Modais:** indicar usuário.
- **Estados vazios:** sem indicações/comissões/saques.
- **Estados de erro:** saque sem saldo.
- **Dados do backend:** indicações, comissões, saldo, saques, níveis.
- **Permissões:** apenas dados do próprio vendedor.
- **Observações de produto:** colapsar 6 abas → 1; a calculadora é marketing, não operação.

---

### Recompensas

- **Objetivo:** gamificação — metas de faturamento com prêmios.
- **Prioridade:** Futuro/Growth.
- **Blocos/cards:** jornada (etapas/metas), progresso (faturado, próxima meta, falta, etapa).
- **Filtros:** N/A.
- **Tabela/colunas:** N/A.
- **Botões/ações:** ação de "próximo passo" ligada ao faturamento real.
- **Modais:** N/A.
- **Estados vazios:** início da jornada (faturado 0).
- **Estados de erro:** N/A.
- **Dados do backend:** faturamento acumulado, metas, prêmios desbloqueados.
- **Permissões:** dados da loja ativa.
- **Observações de produto:** virar **bloco** (Dashboard/Perfil), não tela; ligar a faturamento real.

---

### Conta / Perfil

- **Objetivo:** dados pessoais e preferências de notificação.
- **Prioridade:** MVP (consolidado em "Conta").
- **Blocos/cards:** ficha (avatar, papel, e-mail, status); Dados pessoais; Endereço; Conta (data criação, ID, status conta/documentação — readonly); Preferências (notificações e-mail/WhatsApp).
- **Filtros:** N/A.
- **Tabela/colunas:** N/A.
- **Botões/ações:** Salvar perfil (real); Salvar preferências (real); trocar e-mail/telefone com verificação.
- **Modais:** verificação de contato.
- **Estados vazios:** N/A.
- **Estados de erro:** validação de campos; verificação falha.
- **Dados do backend:** usuário, contatos, preferências, status de conta/documentação.
- **Permissões:** dados do próprio usuário; alterar contato exige verificação.
- **Observações de produto:** unir Perfil + Empresa + Segurança em **uma área "Conta" com abas** (dados se repetem hoje).

---

### Empresa / KYC

- **Objetivo:** cadastro da empresa, representante legal, documentos e compliance.
- **Prioridade:** MVP (libera operação/saque).
- **Blocos/cards:** Informações da empresa (readonly tipo/CNPJ; editável razão/nome comercial/fatura); Representante legal; Detalhes da empresa (faturamento, ticket, produtos, site, contato; toggle "vende físico?"); Compliance externo; **Documentos** (Contrato Social, RG/CNH frente/verso, Selfie com documento) com upload.
- **Filtros:** N/A.
- **Tabela/colunas:** lista de documentos com **status por documento** (aprovado/pendente/recusado).
- **Botões/ações:** upload de documento (real); visualizar; Salvar (real); reenviar documento recusado.
- **Modais:** visualização de documento; reenvio.
- **Estados vazios:** documentos pendentes.
- **Estados de erro:** documento recusado (motivo); arquivo inválido.
- **Dados do backend:** cadastro, documentos, status KYC por documento e geral.
- **Permissões:** **KYC controla liberação de saque/métodos**; somente representante/dono edita.
- **Observações de produto:** status por documento visível é o que falta; KYC é gate de dinheiro.

---

### Segurança / Atividade

- **Objetivo:** proteger a conta (2FA, senha, sessões) e registrar atividade.
- **Prioridade:** MVP.
- **Blocos/cards:** Autenticação (2FA toggle; trocar senha); Sessões ativas (dispositivo, local, hora, encerrar); **Atividade/Auditoria** (absorver a tela Auditoria como aba).
- **Filtros (Atividade):** Tipo de evento, Usuário, Período.
- **Tabela/colunas (Atividade):** Data/Hora, Usuário, Ação, IP, Resultado (Sucesso/Falhou/Atenção).
- **Botões/ações:** ativar 2FA (real); trocar senha (real); encerrar sessão (real).
- **Modais:** confirmação de 2FA (QR/código).
- **Estados vazios:** sem outras sessões; sem eventos.
- **Estados de erro:** senha atual incorreta; 2FA inválido.
- **Dados do backend:** config de segurança, sessões, **log de auditoria** (logins, mudanças sensíveis, saques, chaves).
- **Permissões:** ações sensíveis exigem reautenticação; log imutável.
- **Observações de produto:** fundir **Auditoria** aqui como "Atividade da conta".

---

### Plano e Cobrança

- **Objetivo:** **cobrança da plataforma sobre o vendedor** — mensalidade, taxa por pedido, cartão, faturas, plano contratado. (É o conteúdo atual da tela mal rotulada "Checkout/Cobrança".)
- **Prioridade:** MVP (parte de Conta).
- **Blocos/cards:** Plano atual (mensalidade, taxa por pedido, modelo); Resumo de cobrança (próxima fatura, vencimento, saldo); Dados de cobrança; Cartão em arquivo; Faturas (stats Pendentes/Em atraso/Pagas/Total) + Histórico.
- **Filtros:** período do histórico.
- **Tabela/colunas:** Fatura, Vencimento, Método, Valor, Status, Ações.
- **Botões/ações:** **Alterar plano** (real); **Gerenciar/Editar cartão** (real); Recarregar saldo (real); Ativar loja + checkout (real); Ver fatura (real).
- **Modais:** troca de plano; gerenciar cartões; recarga.
- **Estados vazios:** sem faturas/pendências.
- **Estados de erro:** cobrança recusada; cartão expirado.
- **Dados do backend:** plano, taxas contratadas, faturas, cartão tokenizado, histórico de cobrança.
- **Permissões:** somente dono/financeiro; **NÃO** confundir com Checkout de vendas.
- **Observações de produto:** **renomear para "Plano e Cobrança" e mover para Conta**; remover aba Extrato (duplica Financeiro) e Automação (sem função); liberar o nome "Checkout" para o produto de vendas.

---

## Ações que hoje são `toast()` e precisam virar função real

- **Pedidos:** Exportar; Gerar Relatório; ações em massa (reembolso/marcar enviado).
- **Detalhe do Pedido:** Falar com Suporte / Central de ajuda; Ver item; (reembolso/estorno conforme status).
- **Financeiro:** Exportar (Saques/Extrato); Limpar filtros; Solicitar alteração de taxas; comprovante de saque.
- **Produtos:** + Novo Produto; Copiar link de pagamento; Copiar link de afiliado.
- **Clientes:** + Adicionar Cliente (rever necessidade).
- **Assinaturas:** Ver (detalhe); pausar/cancelar/reativar; dunning.
- **Disputas / Infração:** Anexar arquivos; Enviar defesa; Histórico de transações; Infrações desse documento; Atualizar.
- **Recuperação de Vendas:** Salvar recuperação; toggles de serviço; Saiba mais.
- **Integrações:** Criar webhook (persistir); Criar chave (persistir); Regenerar credenciais; Revelar segredo; Documentação; Adicionar/remover IP; Salvar permissões.
- **Marketplace:** Filtros limpos; afiliar; copiar link de afiliado.
- **Partners:** Copiar link; Indicar usuário; Solicitar saque; Como funciona.
- **Empresa:** Salvar dados; upload de documentos.
- **Perfil:** Salvar perfil; Salvar preferências.
- **Segurança:** Salvar; Encerrar sessão; ativar 2FA.
- **Plano e Cobrança:** Alterar plano; Gerenciar/Editar cartão; Recarregar; Ativar loja + checkout; Ver fatura.

---

## Dados mockados que precisam virar backend real

- **Transação/Pedido:** id, uuid, e2e, cliente, produto, status, método, parcelas, valor bruto/líquido, taxa, custo de adquirência, comissão, lucro, adquirente, ids externos, datas, IP, metadados, rastreio, tentativas de processamento, timeline de status.
- **Cliente:** id, nome, e-mail, CPF/CNPJ, telefone, endereço, agregados (gasto, nº pedidos, ticket, recência), histórico de transações.
- **Produto:** id, nome, tipo, preço, status, estoque, link de pagamento, afiliação/comissão, agregados de venda.
- **Financeiro:** saldos (disponível, a liberar, reservado, bloqueado), limites diários/noturnos, recebíveis antecipáveis, extrato de movimentos, saques (status/datas), tabela de taxas e prazos, chaves Pix, índices de saúde (chargeback/MED/estorno).
- **Assinatura:** id, cliente, plano/produto, valor, método, status, ciclos/cobranças, próxima cobrança, MRR/churn, inadimplência.
- **Disputa/Infração:** id, tipo (MED/Chargeback), status, datas (criação/limite), valor, documentos pagador/recebedor, transação, anexos, defesas, log de webhooks.
- **Checkout (vendas):** checkouts, ofertas, order bumps, upsells, links, métricas de conversão.
- **Recuperação:** serviços ativos por loja, eventos, custos, resultados.
- **Integrações:** webhooks + eventos + log de entregas; chaves de API (escopo/ambiente/IP/uso); credenciais BaaS; catálogo de apps.
- **Conta:** usuário, empresa/KYC + documentos e status, segurança (2FA/sessões), log de auditoria, plano/faturas/cartão.
- **Growth:** marketplace (catálogo/afiliações), partners (indicações/comissões/saques), recompensas (faturamento/metas).

---

## Regras críticas de segurança

- **Autorização por vendedor/loja:** toda query filtra pela loja ativa do usuário autenticado. Troca de loja muda o escopo; nunca vazar dados entre lojas/membros.
- **IDOR:** todo acesso por ID (pedido, cliente, disputa, assinatura, fatura, produto, chave, webhook) valida **propriedade** no servidor — nunca confiar no ID do cliente. Endpoints de detalhe rejeitam IDs de outras lojas.
- **Dados financeiros:** valores e saldos calculados no backend; cliente nunca envia valor a pagar/receber. Operações financeiras são idempotentes e auditadas.
- **KYC:** gate obrigatório — saque, antecipação e habilitação de métodos só com KYC aprovado. Status por documento controlado no servidor; documentos com acesso restrito e expirável.
- **Pix:** chave Pix de saque validada e vinculada ao titular (CNPJ/CPF da conta); mudança de chave dispara verificação e log.
- **Saques:** exigem **2FA**, respeitam **IP whitelist**, limites diários/noturnos e saldo disponível; estado consistente (sem duplo saque); confirmação obrigatória; comprovante auditável.
- **Webhooks:** endpoints só HTTPS; payload assinado (HMAC) para o cliente verificar; segredo de assinatura por webhook; log de entregas com retry; eventos por escopo.
- **API keys:** segredo exibido **uma única vez**; ambientes separados (live/sandbox); rotação/revogação imediatas; escopo mínimo por chave; IP whitelist para saque/estorno; criação/rotação exigem 2FA.
- **Logs/auditoria:** registrar logins, mudanças sensíveis (senha, 2FA, chave Pix, chaves de API, IP, saques, reembolsos) com usuário, IP, timestamp e resultado; log **imutável** e retido.
- **Reautenticação:** ações sensíveis (saque, troca de chave, regenerar credencial, encerrar sessões) pedem reautenticação/2FA recente.
- **LGPD:** dados pessoais de clientes protegidos por permissão; exportações registradas.

---

## Ordem recomendada para o dev

1. **Pedidos** — lista + filtros (essenciais visíveis, resto em "Mais filtros") + **Detalhe do Pedido** + Exportar/Relatório reais + ações em massa. É o núcleo transacional; todas as outras telas referenciam pedidos.
2. **Financeiro** — saldos e Saúde da conta; Saque (2FA + IP whitelist + limites); Antecipação (lote); Extrato (export real); Taxas; Chave Pix. Caixa do vendedor.
3. **Dashboard** — KPIs enxutos + gráfico + cards Top Produtos e Recuperação. Depende de Pedidos/Financeiro já existirem.
4. **Produtos** — CRUD real, link de pagamento funcional, default "Meus produtos", métrica por produto, filtro de status.
5. **Clientes** — lista enxuta + filtros (período/valor/recorrência) + **Detalhe do Cliente** ligado a Pedidos (métricas reais).
6. **Assinaturas** — KPIs + detalhe real (cobranças, pausar/cancelar/reativar) + dunning. Remover duplicata no menu.
7. **Disputas** — lista com contador de prazo + **Detalhe/MED** com defesa e anexos reais; alerta de prazo no Dashboard; tirar "Em construção".
8. **Integrações / API / Webhooks** — webhooks (eventos + log de entregas + teste), chaves de API (ambiente/escopo/IP/2FA), App Store; achatar abas.
9. **Conta** — consolidar Perfil + Empresa/KYC + Segurança/Atividade (absorve Auditoria) + **Plano e Cobrança** (renomeado, movido de "Checkout/Cobrança"). Liberar o nome "Checkout".
10. **Growth** — Checkout de Vendas e Recuperação de Vendas (Fase 3); depois Marketplace, Partners, Recompensas (corrigir stats; colapsar abas; virar bloco).

**Lógica:** primeiro o que **registra dinheiro** (Pedidos → Financeiro → Dashboard), depois o que **gera venda** (Produtos → Clientes → Assinaturas), depois **protege** (Disputas), **conecta** (Integrações), consolida **conta** e por fim **produto de vendas + growth**.

---

## Resumo do que foi criado

- Documento **`PROTOTYPE-IMPLEMENTATION-SPEC.md`** (esta spec), derivado de `PROTOTYPE-PRODUCT-AUDIT.md` e do `index.html` (referência de leitura — **não editado**).
- Conteúdo: como usar; **priorização** (MVP / Fase 2 / Fase 3 / Futuro-Growth); **spec por tela** (20 telas) com objetivo, prioridade, blocos, filtros, colunas, botões, modais, estados vazios/erro, dados de backend, permissões e observações; lista de **ações `toast()` → reais**; lista de **mock → backend**; **regras críticas de segurança** (autorização/IDOR/financeiro/KYC/Pix/saques/webhooks/API keys/logs); **ordem de implementação** detalhada (1–10).
- Distinção de conceito preservada: **Checkout (produto de vendas, Operação)** ≠ **Plano e Cobrança (cobrança da plataforma, Conta)**.
- Nenhum código alterado; `index.html` permanece intacto.
