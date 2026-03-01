# Padrões de Automação — Browser Tool do OpenClaw

Referência técnica para automação de compras via browser tool do OpenClaw.

## Operações Básicas

### Navegação
```
browser(action="navigate", targetUrl="https://ifood.com.br/mercados")
```

### Capturar Estado da Página (substitui screenshot)
```
browser(action="snapshot")
```
Retorna a árvore de acessibilidade da página com refs para cada elemento interativo.

### Clicar em Elemento
```
browser(action="act", request={kind:"click", ref:"<ref>"})
```

### Digitar em Campo
```
browser(action="act", request={kind:"type", ref:"<ref>", text:"termo de busca"})
```

### Pressionar Tecla
```
browser(action="act", request={kind:"press", ref:"<ref>", key:"Enter"})
```

### Hover
```
browser(action="act", request={kind:"hover", ref:"<ref>"})
```

### Preencher Campo (limpa antes)
```
browser(action="act", request={kind:"fill", ref:"<ref>", text:"novo valor"})
```

### Executar JavaScript (evaluate)
```
browser(action="act", request={kind:"evaluate", fn:"return document.title"})
```

### Perfil Recomendado
Use `profile="openclaw"` para browser isolado, evitando interferir no browser do usuário.

## Login no iFood

### Fluxo de Login com Celular

```
1. browser(action="navigate", targetUrl="https://www.ifood.com.br/entrar")
2. browser(action="snapshot")  → encontre botão "Celular"
3. browser(action="act", request={kind:"click", ref:"<celular_ref>"})
4. browser(action="snapshot")  → encontre campo de telefone
5. browser(action="act", request={kind:"type", ref:"<phone_ref>", text:"11999999999"})
6. browser(action="act", request={kind:"press", ref:"<phone_ref>", key:"Enter"})
7. → Pedir código ao cliente (chega por WhatsApp/SMS)
8. → Digitar cada dígito em campo separado (6 campos individuais)
9. Se pedir email: explicar que é verificação de segurança, pedir email ao cliente
```

**Nota:** O número é sem DDD internacional (ex: `11999999999`, não `+5511999999999`).

Os 6 campos de código são inputs individuais — cada dígito vai em um campo separado.

## Extração de Fotos de Produtos

**NUNCA use screenshots da página inteira** — ficam ilegíveis no chat.

### Extrair URLs de Imagens via DOM

```javascript
// Via evaluate
[...document.querySelectorAll('img')]
  .filter(i => i.src.includes('ifood-static') && !i.src.includes('placeholder') && i.naturalWidth > 100)
  .map(i => ({alt: i.alt, src: i.src.replace('t_low','t_high')}))
```

- Substitui `t_low` por `t_high` para imagem de melhor qualidade
- Filtra placeholders e imagens pequenas (ícones, logos)
- Muitos produtos simples não têm foto real — avisar o cliente

## iFood Mercado (ifood.com.br/mercados)

### Descobrir Mercados Disponíveis

```
1. browser(action="navigate", targetUrl="https://ifood.com.br/mercados")
2. browser(action="snapshot")  → identifique cards de mercados
3. Para cada mercado, extraia: nome, avaliação, distância, tempo, taxa
4. Se precisar ver mais: browser(action="act", request={kind:"press", key:"End"})
5. browser(action="snapshot")  → capturar mercados adicionais
```

### Buscar Loja Específica

```
1. browser(action="snapshot")  → encontre ref do campo de busca
2. browser(action="act", request={kind:"click", ref:"<search_ref>"})
3. browser(action="act", request={kind:"type", ref:"<search_ref>", text:"Prezunic"})
4. browser(action="snapshot")  → veja sugestões
5. browser(action="act", request={kind:"click", ref:"<suggestion_ref>"})
```

### Entrar na Loja

Após clicar na loja, a URL muda para:
```
ifood.com.br/delivery/{cidade}/{slug-da-loja}/{id-da-loja}
```

Para acessar diretamente uma loja já conhecida:
```
browser(action="navigate", targetUrl="https://ifood.com.br/delivery/{cidade}/{slug}/{id}")
```

### Buscar Produto

```
1. browser(action="snapshot")  → encontre ref do campo "Busque nesta loja por item"
2. browser(action="act", request={kind:"click", ref:"<search_ref>"})
3. browser(action="act", request={kind:"type", ref:"<search_ref>", text:"arroz"})
4. browser(action="act", request={kind:"press", ref:"<search_ref>", key:"Enter"})
5. browser(action="snapshot")  → veja resultados
```

Para busca seguinte (limpar campo):
```
1. browser(action="act", request={kind:"fill", ref:"<search_ref>", text:"feijão"})
2. browser(action="act", request={kind:"press", ref:"<search_ref>", key:"Enter"})
```

### Adicionar ao Carrinho

**Fluxo A — Direto pelo "+" no card:**
```
1. browser(action="snapshot")  → identifique ref do botão "+" do produto
2. browser(action="act", request={kind:"click", ref:"<plus_ref>"})
3. browser(action="snapshot")  → confirme que card mudou para [-] 1 [+]
```

**Fluxo B — Via página de detalhe:**
```
1. Se clique abriu página de detalhe (URL com ?item=):
2. browser(action="snapshot")  → encontre botão "Adicionar"
3. browser(action="act", request={kind:"click", ref:"<add_ref>"})
4. browser(action="snapshot")  → confirme adição
```

### Aumentar Quantidade

```
1. browser(action="snapshot")  → encontre ref do "+" ao lado do número
2. browser(action="act", request={kind:"click", ref:"<plus_ref>"})
   → repita N-1 vezes para quantidade N
```

### Verificar Carrinho Pré-existente

Ao entrar na loja, sempre faça snapshot e verifique se há carrinho de sessão anterior. Se sim, informe o usuário e peça orientação.

### Monitorar Pedido Mínimo

Calcule `valor_faltante = min_order - total_atual` a cada item adicionado. Informe progresso.

## Navegação do Carrinho (Drawer)

⚠️ **CRÍTICO: O drawer do carrinho trava o Playwright quando se tenta fazer snapshot!**

### Abrir o Drawer

```javascript
// Via evaluate — NÃO use snapshot depois disso
document.querySelector('[data-test-id="header-cart"]')?.click();
```

### Ler Conteúdo do Carrinho

Use evaluate para extrair dados do drawer:

```javascript
// Exemplo: extrair itens do carrinho via DOM
// Adaptar seletores conforme necessário
```

### Ir para o Checkout

O link "Escolher forma de pagamento" é um `<a>` (NÃO button!) com `<span class="btn__label">`:

```javascript
// Via evaluate
const span = [...document.querySelectorAll('span')]
  .find(s => s.textContent === 'Escolher forma de pagamento');
span?.closest('a')?.click();
```

Navega para `https://www.ifood.com.br/pedido/finalizar`.

**Após navegar para a página de checkout, é seguro usar snapshot novamente.**

## Pagamento — Página de Checkout

URL: `https://www.ifood.com.br/pedido/finalizar`

### Selecionar Pix

```javascript
// Pix aparece como button.payment-card
// Clicar para selecionar (fica com classe payment-card--selected)
document.querySelector('button.payment-card')?.click();
// Verificar seleção:
document.querySelector('button.payment-card--selected') !== null
```

Após selecionar Pix, o botão "Fazer pedido" fica habilitado:

```javascript
// Botão tem classe checkout-payment__submit
const btn = document.querySelector('.checkout-payment__submit');
// btn.disabled === false quando pagamento está selecionado
```

### Cupons

Cupons ficam no painel lateral direito como radio buttons:

```javascript
// Listar cupons disponíveis
[...document.querySelectorAll('input[name="checkoutVoucher"]')]
  .map(input => ({
    id: input.value,
    label: input.closest('label')?.textContent?.trim()
  }))
```

⚠️ Muitos cupons do Clube iFood **requerem cartão cadastrado** — não funcionam com Pix.

### Fazer Pedido

**Só com autorização explícita do cliente!**

```javascript
// Clicar em "Fazer pedido"
document.querySelector('.checkout-payment__submit')?.click();
```

### Confirmar Entrega (Modal)

Após clicar "Fazer pedido", aparece modal de confirmação:

```javascript
// Clicar em "Confirmar e fazer pedido"
const confirmBtn = [...document.querySelectorAll('button')]
  .find(b => b.textContent?.includes('Confirmar e fazer pedido'));
confirmBtn?.click();
```

Redireciona para `/pedidos/aguardando-pagamento/{id}`.

### Extrair Código Pix

Na página de aguardando pagamento:

```javascript
// O código Pix começa com "00020101"
const pixCode = [...document.querySelectorAll('*')]
  .map(el => el.textContent?.trim())
  .find(t => t?.startsWith('00020101'));
```

- Enviar o código ao cliente para pagamento
- Cliente tem **~7 minutos** para pagar

## Prezunic (prezunic.com.br) — VTEX

### Buscar Produto

URL direta:
```
browser(action="navigate", targetUrl="https://www.prezunic.com.br/arroz%20tio%20joao?_q=arroz%20tio%20joao&map=ft")
```

Via campo de busca:
```
1. browser(action="snapshot")  → encontre campo de busca
2. browser(action="act", request={kind:"type", ref:"<search_ref>", text:"arroz tio joao"})
3. browser(action="act", request={kind:"press", ref:"<search_ref>", key:"Enter"})
```

### Adicionar ao Carrinho

```
1. browser(action="snapshot")  → encontre botão "COMPRAR" do produto
2. browser(action="act", request={kind:"click", ref:"<buy_ref>"})
3. browser(action="snapshot")  → confirme adição (se abriu detalhe, clique "Adicionar ao carrinho")
```

## Estratégia de Busca Progressiva

```
Tentativa 1: "Feijão Preto Super Máximo 1Kg"  → 0 resultados
Tentativa 2: "Feijão Preto Super Máximo"       → 0 resultados
Tentativa 3: "Feijão Preto"                    → 8 resultados ✓
```

## Registro de Preços

Após cada item adicionado, registre:
- Produto (nome completo como aparece)
- Marca, Tamanho, Preço unitário, Quantidade, Subtotal
- Nota (substituição? promoção?)

Compile relatório ao final com total estimado.
