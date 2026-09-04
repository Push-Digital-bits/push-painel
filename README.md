# Radar de Campanhas & Chamados — contexto do projeto

Documento de referência. Serve para que qualquer conversa nova já comece
sabendo como o sistema foi montado, sem precisar refazer as decisões.

Última atualização: setembro de 2026.

---

## O que é

Painel único para a agência **Push Digital**, com duas seções:

- **Radar de Campanhas** — acompanha o prazo de cada campanha/oferta, por
  loja e por plataforma (TikTok, Shopee etc.), com status "no prazo / vence
  em breve / vencida" e saldo de anúncios (orçamento total menos gasto por
  dia).
- **Chamados** — chamados internos por loja, com lista de tarefas, nível de
  urgência (1 a 4) e prazo de entrega.

Tem também um **Modo Dashboard** (tela cheia, pensado pra TV) que mostra
Campanhas e Chamados lado a lado.

Antes esse painel vivia como um Artifact do Claude. Foi portado pra essa
arquitetura (igual ao Estoque Domah) porque o Artifact só permite edição
compartilhada entre contas Claude da mesma organização — pra qualquer
pessoa poder abrir o link e editar, sem precisar de conta Claude, o jeito é
esse: planilha + GitHub Pages.

## Arquitetura

Três peças, cada uma com um papel:

| Peça | Onde vive | Função |
|---|---|---|
| Planilha Google Sheets | Google Drive | O cérebro. Guarda lojas, campanhas e chamados |
| Apps Script | Dentro da planilha | A porta que deixa gravar. Publicado como App da Web |
| Painel `index.html` | GitHub Pages | A vitrine. Lê e escreve pela porta acima |

O painel fala com o Apps Script por `POST` com `Content-Type: text/plain`,
o que evita o preflight de CORS. A senha vai no corpo de cada pedido.

Como não é mais um banco em tempo real (era assim no Artifact, via
capacidade `db`), o painel busca a planilha de novo sozinho a cada 25
segundos — assim quem estiver com o link aberto vê o que outra pessoa
mudou, sem precisar recarregar a página.

## Endereços

- **Planilha:** https://docs.google.com/spreadsheets/d/1TM8DkqrQrxckLBd2VNn59JFuISZKUT6VBwkM8M7Ixw0/edit
- **Painel no ar:** https://push-digital-bits.github.io/push-painel/
- **Repositório:** github.com/Push-Digital-bits/push-painel
- **Conta GitHub:** Push-Digital-bits

A senha do script fica no `Codigo.gs` e no `config.js`. Ela não é secreta:
o repositório é público e o arquivo é legível por qualquer um. Serve só
para evitar que alguém que tropece na URL do script mexa no painel sem
querer.

## Arquivos

- `Codigo.gs` — vai colado no Apps Script da planilha
- `index.html` — o painel. Descartável: pode ser trocado inteiro. A marca da
  Push Digital vai embutida nele (base64), não é arquivo separado
- `config.js` — guarda URL e senha. **Nunca é substituído**

`index.html` e `config.js` ficam juntos na raiz do repositório.

## Estrutura da planilha

**Aba `Lojas`:** id, nome, logo (link da imagem)

**Aba `Campanhas`:** id, loja, plataforma, prazo, nota, arquivada,
orcamentoTotal, gastoDiario, criadoEm

**Aba `Chamados`:** id, loja, titulo, tarefas (JSON), urgencia, prazo,
status, concluidoEm, criadoEm

Datas ficam guardadas como texto ISO puro (nunca como Date da planilha),
pra não depender do idioma/fuso da planilha — mesma lição aprendida no
Estoque Domah com a coluna de saldo.

Quando uma campanha ou chamado é criado pra uma loja que ainda não tinha
linha própria em `Lojas`, o script cria a linha sozinho (sem logo). Isso
permite criar tudo pelo chat, sem precisar passar por "+ Nova loja" antes.

## Diferenças em relação à versão Artifact

- **Logo da loja:** antes era upload de arquivo (virava imagem embutida,
  base64). Agora é um link de imagem colado — igual ao Estoque Domah — pra
  não estourar o limite de caracteres de uma célula da planilha (~50.000).
  Clique no avatar da loja no card pra colar/trocar o link.
- **Preenchimento por voz:** já vinha desativado no Artifact (o microfone
  não é liberado dentro de artefatos publicados) e continua desativado
  aqui — o código ficou no arquivo mas nunca é ativado.
- **Sincronização:** no Artifact era em tempo real (banco `db`). Aqui é
  busca automática a cada 25s.

## Como atualizar

Mudança no comportamento (`Codigo.gs`):
1. Colar o código novo em Extensões → Apps Script, salvar
2. Implantar → Gerenciar implantações → lápis → **Nova versão** → Implantar

O passo 2 é o mais esquecido. Sem ele, o Google continua servindo o
código antigo e nada muda no painel.

Mudança no visual (`index.html`): editar o arquivo direto no GitHub (ícone
de lápis na página do arquivo) ou subir um novo via Add file → Upload
files. Em dois minutos está no ar. O `config.js` fica intacto.

## Armadilhas já encontradas (herdadas do Estoque Domah, evitadas aqui)

- **Logo embutida em base64 estoura célula/arquivo.** Por isso o logo da
  loja é link, não upload.
- **Esquecer de republicar o Apps Script** faz parecer que o código novo
  não funcionou.
- O repositório é público, então tudo que está nele é legível.

## O que ficou pendente

- Automação de vendas do TikTok Shop pra alimentar campanhas sozinho:
  ainda sem solução (ver decisões anteriores do projeto — o conector
  AfterShip Feed não serve pro caso de uso).
