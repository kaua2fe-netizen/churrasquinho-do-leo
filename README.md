# Churrasquinho do Léo — Plataforma de delivery e gestão

Next.js 15 (App Router) + TypeScript + Tailwind CSS + Supabase (Auth, PostgreSQL, RLS).

## Estado atual (honesto)

| Etapa | Conteúdo | Situação |
|---|---|---|
| 1 | Banco: perfis, permissões, cardápio, pedidos, auditoria, RLS, seed | Executada por você no Supabase com sucesso |
| 2 | Projeto Next.js, autenticação, tema, home, cardápio, carrinho, base do painel | **Código escrito; ainda NÃO compilado nem testado** (ver abaixo) |
| 3 | Checkout (preço recalculado no servidor), pagamento, endereços, acompanhamento em tempo real | Pendente |
| 4+ | PDV, caixa, mesas, cozinha, entregas, estoque, compras, financeiro, relatórios, CRM, etc. | Pendente |

### O que a Etapa 2 entrega
- Conexão com Supabase: cliente de navegador (chave anon), cliente de servidor (sessão do usuário) e cliente admin (service role, só servidor, protegido por `server-only`).
- Login, cadastro, recuperação e redefinição de senha para clientes; login separado e restrito em `/painel-restrito/login`.
- Proteção de rotas em duas camadas: `middleware.ts` e verificação no servidor (`exigirEquipe()`), que espelha as regras do banco.
- Home, cardápio por categorias com busca, carrinho persistente (localStorage), verificação de maioridade para bebidas alcoólicas, tema claro/escuro, estados de carregamento e de erro.
- Painel: dashboard com contagens reais (pedidos pendentes/hoje, produtos ativos/esgotados) e página **Produtos** (editar preço, ativo e esgotado; reflete no site e entra na auditoria).
- Módulos ainda não construídos aparecem no menu como "em breve" (não clicáveis).

### O que ainda não existe (não finja que existe)
- Checkout: o botão "Finalizar pedido" está desativado de propósito.
- Fotos reais dos produtos: `imagem_url` está vazio no seed; hoje aparece uma imagem alternativa. Envie as fotos pelo Supabase Storage ou me passe os arquivos.
- 2FA (MFA) no painel: ative no Supabase; a tela de verificação será feita numa etapa futura.
- Limite de tentativas de login: por enquanto vale o rate limit do próprio Supabase Auth. Histórico de acessos ainda não existe (não há tabela para isso no SQL da Etapa 1).
- Avaliações, promoções e "mais vendidos" na home: dependem de dados que ainda não existem, então não são exibidos.

## Instalação

Requisitos: Node.js 20+.

```bash
npm install
cp .env.example .env.local     # no Windows: copy .env.example .env.local
# preencha .env.local (veja abaixo)
npm run typecheck
npm run dev                    # http://localhost:3000
```

### .env.local
- `NEXT_PUBLIC_SUPABASE_URL` e `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Supabase > Project Settings > API.
- `SUPABASE_SERVICE_ROLE_KEY`: mesma tela. **Secreta**: nunca publique nem use no navegador. Ainda não é usada pelas telas da Etapa 2 (será no checkout).
- As demais variáveis (WhatsApp, PIX) são para etapas futuras.

### Configurar autenticação no Supabase
Authentication > URL Configuration:
- Site URL: `http://localhost:3000` (depois, o domínio real).
- Redirect URLs: `http://localhost:3000/auth/callback` e `https://SEU-DOMINIO/auth/callback`.

Authentication > Providers > Email: mantenha "Confirm email" ligado em produção. Para testar rápido em desenvolvimento, pode desligar.

### Primeiro proprietário
Siga o procedimento da Etapa 1: crie o usuário em Authentication > Users e rode uma única vez no SQL Editor:
`update perfis set papel = 'proprietario' where id = (select id from auth.users where email = 'EMAIL_DO_PROPRIETARIO');`
Depois acesse `/painel-restrito/login`.

### Roteiro de testes manuais (faça antes de avançar)
1. Home e cardápio mostram as 3 categorias e todos os itens do seed, com preços corretos.
2. Adicione itens, mude quantidades, escreva observação, recarregue a página: o carrinho permanece.
3. Adicione uma cerveja: aparece a pergunta de maioridade.
4. No painel, mude o preço de um produto e confira no cardápio; confira também a tabela `auditoria`.
5. Cadastre um cliente, confirme o e-mail, faça login/logout e teste "Esqueci a senha".
6. Logue com um cliente em `/painel-restrito/login`: deve ser recusado ("Acesso não autorizado").
7. Acesse `/painel-restrito` sem login: deve ir para o login.
8. Teste no celular (navegação inferior).

## Publicação na Vercel
1. Suba o projeto para um repositório Git e importe na Vercel.
2. Cadastre as variáveis de ambiente (as mesmas do `.env.local`) em Project Settings > Environment Variables.
3. Faça o deploy e, no Supabase, inclua o domínio da Vercel em Site URL e Redirect URLs.
4. Domínio próprio: Vercel > Domains; depois atualize as URLs no Supabase.
