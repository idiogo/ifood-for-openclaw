# ifood-for-openclaw

OpenClaw skill para automação de compras e pedidos no iFood (Mercado + Delivery).

## Sobre

Skill que transforma o OpenClaw em um assistente inteligente de compras de supermercado e pedidos em restaurantes, com automação de navegador para iFood Mercado, iFood Delivery e outros mercados online. Funcionalidades:

- 🛒 Carrinho recorrente baseado em histórico
- 🍔 Pedidos em restaurantes via iFood Delivery
- 🔄 Substituições inteligentes de produtos
- 💰 Comparação multi-loja de preços
- 📋 Planejamento semanal de refeições → lista de compras
- 💵 Controle de orçamento com alertas
- 🏷️ Alertas de queda de preço
- 💳 Checkout completo com Pix (com autorização do cliente)
- 🔐 Login automatizado via celular + código de verificação

## Instalação

```bash
git clone https://github.com/idiogo/ifood-for-openclaw.git
cp -r ifood-for-openclaw/skills/ifood ~/.agents/skills/
```

## Créditos

Baseado no projeto [assistente-compras-ifood](https://github.com/fabianocruz/assistente-compras-ifood) por **Fabiano Cruz**, originalmente criado como skill para Claude Cowork. Adaptado para o ecossistema OpenClaw com browser tool nativo.

## Licença

MIT — veja [LICENSE](LICENSE).
