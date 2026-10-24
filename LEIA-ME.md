# Inventário — App de Contagem de Estoque

App de contagem para açougues, frigoríficos e centros de distribuição. Roda em **Android, iPhone e coletores de dados** pelo navegador, funciona **100% offline** e já vem com os **360 produtos** da planilha `GRUPO01 Produtos Terneiro.xlsx` cadastrados.

## Arquivos

| Arquivo | O que é |
|---|---|
| `index.html` | O app inteiro (interface, banco de dados, importação e exportação) |
| — | usa SheetJS para ler planilhas e ExcelJS para gerar a planilha formatada, ambos por CDN e guardados no cache offline na primeira abertura |
| `manifest.json` | Faz o app instalar na tela inicial como aplicativo |
| `sw.js` | Cache offline — funciona sem sinal dentro da câmara fria |
| `gerador-de-chaves.html` | **Só para você.** Emite chaves de liberação. Nunca distribua este arquivo. |
| `COMO-ABRIR-NO-IPHONE.md` | Passo a passo para publicar no GitHub Pages e instalar no iPhone |

## Como colocar em uso

**Opção 1 — hospedar (recomendado, libera instalação e offline)**

Suba as três pastas/arquivos em qualquer hospedagem estática gratuita: GitHub Pages, Netlify, Vercel ou a pasta pública do seu servidor. Depois é só abrir o endereço no celular ou coletor e usar **Menu → Adicionar à tela inicial**. A partir daí o app abre em tela cheia, sem barra de navegador, e funciona sem internet.

**Opção 2 — testar rápido no computador**

```bash
cd pasta-do-app
python -m http.server 8080
```
Abra `http://localhost:8080`. No celular na mesma rede Wi-Fi, use `http://IP-DO-PC:8080`.

> Abrir o `index.html` com duplo clique (`file://`) **não funciona** na maioria dos navegadores — eles bloqueiam o banco de dados local. Use sempre um endereço `http` ou `https`.

## Trava de uso: chave de liberação

O app só funciona em aparelhos liberados por uma chave. Sem ela, a tela de contagem nem abre.

**Sua chave:**

```
TRN-4TF6N-6SS76-RNVAE-WVV5P
```

Licenciada para *Terneiro — Açougue e Frigorífico*, válida até **31/12/2027**. Guarde num lugar seguro e entregue só a quem for operar o inventário. Em cada aparelho, informe a chave uma única vez: ela fica gravada e o app não pede de novo.

### O que isso protege, e o que não protege

O arquivo **não contém a chave** — só a impressão digital dela (um hash SHA-256). Quem abrir o `index.html` num editor de texto não encontra a chave em lugar nenhum, nem consegue deduzi-la. Então copiar o arquivo e levar para outra empresa não adianta: sem a chave, o app fica travado na tela de liberação.

Sendo honesto sobre o limite: toda a verificação acontece dentro do próprio arquivo, no aparelho. Alguém com conhecimento técnico, tempo e o arquivo em mãos consegue editar o código e remover a trava. Isso não é defeito da implementação, é a natureza de um app que roda offline sem servidor. O que está aqui **impede o uso casual não autorizado** — alguém que recebeu o arquivo por WhatsApp e quis usar na própria loja. Não impede um programador determinado. A única defesa contra esse caso seria validar a licença num servidor, o que exige internet e hospedagem.

Por isso a **validade** importa: mesmo que uma chave vaze, ela morre na data. E o nome do licenciado aparece no cabeçalho do app e na primeira linha de toda planilha exportada, o que deixa rastro em qualquer cópia que circule.

### Quando a licença vence

O app entra em **modo limitado**, e isso foi desenhado para não te deixar na mão: contar fica bloqueado, mas a tela de liberação oferece **Exportar minhas contagens** e **Gerar backup completo**. Nada do que já foi contado se perde. Trinta dias antes do vencimento, um aviso aparece na própria tela de contagem.

Atrasar o relógio do aparelho não reabre uma licença vencida — o app guarda a data mais adiantada que já viu e usa a maior das duas.

### Emitir novas chaves

Abra o `gerador-de-chaves.html` no navegador (funciona offline, nada sai do seu computador):

1. Preencha o nome do licenciado e a validade.
2. Clique em **Gerar chave**. Aparecem duas coisas: a **chave**, para entregar a quem vai usar, e uma **linha de código**.
3. Abra o `index.html` num editor de texto, procure por `const LICENCAS = [` e cole a linha dentro dos colchetes.
4. Salve e distribua o `index.html` atualizado.

Use chaves diferentes por unidade ou por safra de inventário: se uma vazar, você revoga só aquela, removendo a linha correspondente do arquivo. Aparelhos que usavam a chave removida passam a exibir "chave não reconhecida" e pedem uma nova — as contagens deles continuam guardadas.

Em **Ajustes → Licença de uso** você vê para quem o aparelho está licenciado, até quando, e pode **remover a ativação** ao aposentar ou vender um aparelho.

## Fluxo de contagem

1. Informe o nome do operador (é pedido na primeira abertura e fica gravado em cada lançamento).
2. Na aba **Contar**, o cursor já está no campo de busca.
3. Digite o código, digite o **nome**, ou dispare o leitor — o produto aparece na hora.
4. Digite a quantidade (KG com decimais, UN inteiro) ou use os botões rápidos.
5. Nos produtos por peso, **Enter** pula para o campo **Peças**, já preenchido com 1.
6. **Enter** de novo grava — e o produto **continua na tela** para a próxima pesagem.

Atalhos: `Enter` avança e grava · `Esc` solta o produto e volta ao código · botões `+0,5 / +1 / +2…` somam sem digitar · `⌫` apaga o último dígito.

## Produto fixo na tela

Vem ligado por padrão. Depois de gravar, o produto **não sai da tela**: o campo de peso é limpo, as peças voltam para 1 e o cursor já está lá. Você pesa a bandeja, Enter, Enter, pesa a próxima — sem redigitar o código nenhuma vez.

O painel do item mostra o acumulado em tempo real: o total geral daquele produto no inventário e, embaixo, `📌 Nesta sessão: 7 pesagens · 84,320 KG · 12 pç`.

Para trocar de produto há três caminhos:

- **Ler outro código de barras**, mesmo com o cursor no campo de peso. O app reconhece que foi o leitor (leitores digitam a rajada inteira em menos de 45 ms por caractere, coisa que ninguém faz na mão), apaga o que caiu no campo errado e abre o novo item sozinho. Digitação humana, por mais rápida que seja, nunca é confundida com leitura.
- **`Esc`** ou o botão **🔄 Trocar de produto**, que devolve o cursor ao campo de código.
- Desligar o modo no botão **📌 PRODUTO FIXO** dentro do card, ou em Ajustes. Aí o app volta ao comportamento de pedir o código a cada item.

## Quantidade de peças

Nos produtos vendidos por **KG**, ao lado do peso há um campo **Peças**: quantas peças formaram aquele peso. Uma bandeja de 12,345 KG com 3 cortes é gravada como `12,345 KG · 3 pç`. O campo já vem com `1`, então quem não precisa desse controle só dá Enter e segue — não atrasa a contagem.

Nos produtos por **UN**, a quantidade digitada já é a própria contagem de peças, então não existe campo duplicado.

O total de peças aparece no relatório (separado entre KG e UN), na lista de produtos, no histórico e em duas colunas da planilha exportada: `Qtd. Peças` no resumo e `Peças` no histórico. Se preferir desativar, é o botão *Contar peças nos itens KG* em Ajustes — nesse caso todo lançamento em KG grava 1 peça.

O código pode ser digitado **sem os zeros à esquerda** — digitar `112` encontra `0000000000112`. Códigos de 13 dígitos disparam a busca automaticamente, sem precisar de Enter, para leitores que não enviam o "Enter" no fim.

## Buscar por nome e escolher entre parecidos

O campo de código aceita **nome também**. A partir de duas letras, aparece logo abaixo a lista dos produtos que combinam, com o trecho digitado destacado, a unidade, a família, o departamento e quanto já foi contado de cada um. Funciona com palavras soltas em qualquer ordem: `FRANGO PEITO` encontra *FILET DE PEITO FRANGO*.

Como escolher:

- **Toque** na opção desejada, ou
- **↑ ↓** para navegar e **Enter** para abrir — tudo pelo teclado, importante nos coletores sem toque.
- **Esc** fecha a lista sem escolher nada.

Quando o que você digitou tem **mais de um candidato** — um nome, ou um pedaço de código que aparece em vários produtos — o Enter não escolhe sozinho: ele destaca a primeira opção e espera a sua confirmação. Nada é aberto por adivinhação.

Já um **código exato** continua abrindo direto, com um único Enter: seja o código completo, seja ele sem os zeros à esquerda. A velocidade da contagem por código não mudou — a lista de parecidos aparece enquanto você digita, então você vê as alternativas antes mesmo de apertar Enter.

## Recursos

**Cadastro pelo próprio app, sem planilha** — código único, nome, unidade (KG/UN), família e departamento. Há duas portas de entrada:

- Aba **Produtos** → botão **＋ Cadastrar novo produto**.
- Durante a contagem, quando um código não é encontrado, aparece **＋ Cadastrar produto** com o código já preenchido. Você completa o nome, salva, e o app abre o item direto na tela de quantidade — sem sair do fluxo, sem importar arquivo nenhum.

Família e departamento têm sugestão automática dos valores já existentes, e o Enter salva de qualquer campo. Para corrigir um item (inclusive trocar KG↔UN), é só tocar nele na aba Produtos.

**Histórico** — cada pesagem é um registro separado, nunca substitui o anterior. Guarda data, hora, produto, código, quantidade, unidade, peças, família, departamento e usuário. Filtros por departamento e por operador.

**Relatório** — percentual de conclusão do inventário, totais em KG, em UN e em peças, progresso por departamento, participação de cada produto no estoque e totais consolidados por produto.

## % do produto no estoque

Somar quilos com unidades daria um número sem significado, então o percentual tem duas bases, ambas na planilha e no app:

- **% da unidade** — a fatia do produto no total da sua unidade. Todos os itens em KG fecham 100% entre si; os itens em UN fecham 100% à parte.
- **% do departamento** — a fatia do produto dentro do próprio departamento, também respeitando a unidade. Serve para enxergar o mix de cada seção: quanto a picanha representa de BOVINOS, por exemplo.

No app isso vira a seção **Participação no estoque**, dentro de Relatório: um ranking com barras, ordenado do maior para o menor, com chips para alternar entre KG e UN e entre as duas bases de cálculo. Cada produto na lista consolidada também mostra as duas porcentagens.

**Exportação (.xlsx)** — planilha formatada, no padrão da `contagem_camara`: título em vermelho, cabeçalhos brancos sobre vinho, linhas de total em preto, zebrado, filtros automáticos, painéis congelados e formatos numéricos por unidade (3 casas em KG, inteiro em UN, percentual com uma casa). São quatro abas:

- **RESUMO** — bloco de cabeçalho (início, fim, responsáveis, total de pesagens, avanço do cadastro, totais gerais) e a tabela: produto, código, família, departamento, UN, pesagens, peças, quantidade total, **% da unidade**, **% do depto.**, média por pesagem e **como foi contado** — a sequência das pesagens na ordem, `200,000 + 100,000 + 500,000 + 150,000`. Fecha com TOTAL EM QUILOS e TOTAL EM UNIDADES.
- **HISTÓRICO POR PRODUTO** — um bloco por item, cada pesagem numerada com data, hora, quantidade, peças e usuário, seguida do total e da média do produto.
- **LANÇAMENTOS** — tudo em ordem cronológica, para auditoria linha a linha, com as colunas **APARELHO** e **ID** que permitem reimportar a planilha sem duplicar.
- **CONSOLIDAÇÃO** — totais por aparelho e por operador, e a lista de produtos contados em mais de um coletor.
- **NÃO CONTADOS** — o que ficou pendente no inventário.

Os códigos saem formatados como texto, então os zeros à esquerda não se perdem ao abrir no Excel.

Também há **backup em .json** com produtos e contagens, para guardar ou transferir.

**Importação** — na aba Ajustes você importa outra planilha a qualquer momento. O app aceita o formato original (blocos `Família:` / `DEPARTAMENTO:` / `Cód. Barra` / `Descrição`) e também planilhas com colunas nomeadas `codigo, nome, unidade, familia, departamento`. Produtos com código já existente são atualizados; o código nunca é alterado.

**Etiquetas de balança** — em Ajustes, ative *Etiquetas de balança*. O app passa a interpretar EAN-13 de peso no padrão `2 + PLU(6) + gramas(5) + DV`, localiza o produto pelo PLU e já preenche o peso em KG. Se sua balança usa outro layout, é só ajustar as posições na função `S.find` do `index.html`.

## Unidade de medida: tudo em KG

**Os 360 produtos são contados por peso.** Toda contagem é em quilos, com três casas decimais, e o campo **Peças** ao lado registra quantas peças formaram aquele peso.

Ao abrir esta versão pela primeira vez, cada aparelho faz a conversão sozinho:

- os produtos que estavam em UN passam para **KG**;
- as contagens anteriores são **apagadas**, para o inventário começar limpo e sem registros com a unidade antiga;
- antes de apagar, o app guarda uma cópia — se precisar do que havia ali, está em *Ajustes → Cópias de segurança internas*;
- um aviso explica o que aconteceu, e isso roda **uma vez só** por aparelho.

Repita a atualização em **todos** os celulares e coletores: é a abertura do novo `index.html` que dispara a conversão em cada um.

Importar uma planilha de produtos sem coluna de unidade agora cadastra tudo como KG. Se um catálogo antigo trouxer itens em UN de volta, Ajustes avisa e oferece **⚖️ Converter todos para KG**. O app continua aceitando UN produto a produto, caso um dia você precise de uma exceção — é só editar o item na aba Produtos.

## Cadastrou um produto num celular? Repasse o catálogo

Cada aparelho tem seu próprio banco. Um produto cadastrado no Coletor 1 não aparece sozinho no Coletor 2 — e é por isso que existe o **catálogo**.

Em **Ajustes → Enviar para outro aparelho → 🏷️ Enviar catálogo de produtos**, o app gera um arquivo pequeno só com os produtos, sem contagem junto. Mande por WhatsApp, e nos outros aparelhos use **📥 Receber arquivo**.

Ao receber um catálogo o app:

- **acrescenta** os produtos que faltavam;
- **atualiza** os que mudaram de nome, unidade, família ou departamento;
- **nunca apaga** um produto que só existe no aparelho de destino;
- ignora o que já está igual, então reenviar o mesmo arquivo não faz mal nenhum.

Para você não esquecer, quando alguém cadastra um produto o aviso aparece em Ajustes: *"⚠️ 3 produtos cadastrados aqui ainda não foram enviados aos outros aparelhos"*. O aviso some depois que o catálogo é gerado.

## Trocar de celular ou mudar o app de endereço

O banco fica preso ao aparelho **e ao endereço**: se você mover o `index.html` para outra pasta, outro domínio ou outro celular, o app abre vazio — os dados não sumiram, eles ficaram no endereço antigo. A ponte é o backup completo.

1. No aparelho antigo: **Ajustes → 📦 Backup completo (produtos + contagens)**.
2. Leve o arquivo para o aparelho novo.
3. Lá: **📥 Receber arquivo** → escolha **🔄 Substituir tudo (mudar de aparelho)**.

Vai junto tudo: produtos, todos os lançamentos, o nome do operador, o nome do aparelho e suas preferências. O aparelho novo mantém apenas o **código interno próprio**, de propósito: assim, se os dois continuarem contando por engano, os lançamentos não colidem e a consolidação ainda funciona.

Se o aparelho de destino já tiver contagens, o app pergunta antes: **somar** ao que existe ou **substituir tudo**. Somar aproveita a mesma deduplicação da consolidação — nada se perde e nada duplica.

> Faça o backup **antes** de mudar o app de lugar. Depois de trocar o endereço, o app novo não enxerga os dados do antigo para exportar.

## Consolidar vários coletores

Cada aparelho tem um **nome** (defina em Ajustes: "Coletor 1 — Câmara fria") e um código interno de 4 caracteres. Todo lançamento sai carimbado com esse nome e com um identificador único.

Para juntar tudo num lugar só:

1. Em cada coletor, gere o **backup .json** (ou a própria planilha .xlsx).
2. Envie os arquivos para o aparelho que vai consolidar — WhatsApp, e-mail, cabo, o que for.
3. Nesse aparelho: **Ajustes → Receber contagens de outro aparelho**. Pode selecionar vários arquivos de uma vez.
4. Exporte a planilha final.

> **Se você consolidou por planilha antes da v1.4**, confira os valores: havia um erro que multiplicava as quantidades por mil ao ler a aba LANÇAMENTOS (500 KG virava 500.000 KG). O backup `.json` nunca foi afetado. Zere as contagens do aparelho consolidador e refaça a importação com esta versão.

O app soma o que falta e **ignora o que já entrou**. Importar o mesmo arquivo duas vezes não duplica nada — o controle é pelo identificador de cada lançamento, não pelo nome do arquivo. Se um coletor continuou contando depois de mandar o primeiro arquivo, basta reenviar: só as pesagens novas entram. Produtos que existem lá e faltam aqui também são cadastrados junto.

A planilha ganha a aba **CONSOLIDAÇÃO**, com totais por aparelho e por operador (lançamentos, produtos, KG, UN, peças e o período trabalhado) e uma seção que lista os **produtos contados em mais de um aparelho** — o app soma todos, mas essa lista existe para você conferir se as áreas não se sobrepuseram.

## Onde ficam os dados e como não perdê-los

Tudo é gravado no IndexedDB do próprio aparelho. Nada trafega pela internet. Aguenta dezenas de milhares de produtos com busca instantânea, porque a localização é feita por índice em memória (Map), não por varredura.

Armazenamento local tem riscos reais, e o app trata cada um:

- **Despejo por falta de espaço** — na abertura o app pede `navigator.storage.persist()`, que marca os dados como persistentes e os tira da fila de descarte do navegador. Ajustes mostra se a proteção foi concedida.
- **Safari apagando tudo após 7 dias** — o WebKit remove armazenamento de sites não visitados nesse prazo, mas **web apps instalados na tela inicial são isentos**: têm armazenamento próprio, isolado do Safari. Por isso, no iPhone, instalar na tela inicial não é conforto, é proteção. Se a persistência não foi concedida, o app avisa em vermelho na tela de contagem.
- **Zerar por engano** — o botão agora exige digitar `ZERAR`, e tira uma cópia antes de apagar.
- **Exclusão ou pane no meio do inventário** — a cada 50 lançamentos, e antes de qualquer operação de risco, o app guarda uma cópia interna. As 5 mais recentes ficam disponíveis em *Ajustes → Cópias de segurança internas*, com restauração em um toque.
- **Esquecer de exportar** — Ajustes mostra quantos lançamentos ainda não saíram do aparelho e a data da última exportação. Passando de 100 pendentes, aparece um aviso na própria tela de contagem.
- **Perder ou quebrar o aparelho** — contra isso não há truque de software. Exporte a planilha ou o backup periodicamente; é a única cópia que sobrevive ao aparelho.

As cópias internas protegem contra erro de operação, não contra alguém limpar os dados do navegador ou o aparelho sumir. **A exportação continua sendo o backup de verdade.**

## Evoluções previstas

A arquitetura separa banco (`DB`), regras (`S`) e telas (`vCount`, `vProds`, …), então dá para evoluir sem reescrever:

- **Sincronização multi-usuário** — trocar a camada `DB` por chamadas a uma API, mantendo o IndexedDB como fila offline.
- **ERP** — o objeto de contagem já tem todos os campos necessários; basta um `POST` no lugar (ou além) do `DB.put`.
- **Câmera como leitor** — encaixa em `lookup()` com uma biblioteca de leitura por vídeo.
- **Balança serial/Bluetooth** — o campo de quantidade recebe o valor pronto, igual ao que a etiqueta de peso já faz hoje.

## Verificação

O app passou por **257 testes automatizados** em nove frentes.

*Exclusão, busca e iOS (35):* confirmação que aparece antes de apagar uma pesagem, mostrando produto, quantidade, peças, operador e o efeito no total, com cancelar preservando o registro e confirmar recalculando os totais; busca por nome trazendo a lista completa (29 itens com "FRANGO", não os 12 de antes) com o contador correto; e o caminho de exportação no iPhone usando a folha de compartilhamento do sistema, com a orientação de salvar em Arquivos quando o compartilhamento não existe, sem alterar o comportamento no computador.

*Trava de licença (32):* o hash SHA-256 próprio conferido contra o do Node em sete casos, incluindo acentuação e entradas longas; garantia de que a chave não aparece em texto no arquivo; aparelho novo travado e sem pedir o operador antes de liberar; recusa de chave inventada, vazia e com um caractere trocado; aceitação da chave em minúsculas, com espaços e sem hífens; licença que persiste na abertura seguinte; vencimento que trava a contagem mas preserva a exportação; relógio atrasado que não reabre licença vencida; contagem dos dias restantes e aviso antecipado; e remoção da ativação.

*Conversão para KG (24):* instalação nova já toda em quilo; aparelho que vinha da versão anterior com produtos em UN e contagens gravadas, conferindo que os produtos convertem, que os lançamentos antigos somem, que nenhum registro em UN sobra, que o operador é preservado e que o apagado fica recuperável nas cópias internas; garantia de que a conversão **não roda duas vezes**; contagem e relatório já em quilo com os percentuais fechando 100%; e o botão manual convertendo um item em UN que entre depois.

*Catálogo e migração (32):* dois aparelhos com bancos realmente separados; produto cadastrado em um que atravessa para o outro pelo catálogo e passa a ser encontrado por código e por nome; catálogo reenviado que não duplica; produto exclusivo do destino que sobrevive ao recebimento; correção de unidade que se propaga; migração completa levando produtos, lançamentos, operador, nome do aparelho e preferências, mantendo código interno próprio; migrar duas vezes sem duplicar; somar que traz o que falta sem apagar o que era do destino; cancelar que não altera nada; e o reconhecimento automático do tipo de cada arquivo.

*Leitura de números:* a conversão de texto para número é testada com os dois padrões (`1.234,567` e `1,234.567`), com decimais simples (`500,5` e `500.5`), com agrupamento (`1.234.567`) e com entradas vazias ou inválidas. A ida e volta pela planilha confere agora **os valores lidos**, não apenas a quantidade de linhas — foi essa lacuna que deixou passar o erro de multiplicação por mil.

*Busca e desambiguação (28):* localizar por nome, por palavra no meio do nome e por palavras em qualquer ordem; ordem dos candidatos com o código exato à frente dos parecidos; lista aparecendo ao digitar; navegação por ↑ ↓ e Enter; `Esc` fechando a lista; nome ambíguo que não abre sozinho e exige confirmação; código exato — completo ou sem zeros — abrindo direto mesmo existindo códigos parecidos; e código inexistente caindo na oferta de cadastro.

*Proteção e consolidação (23):* pedido de persistência na abertura, carimbo de aparelho e identificador único em cada lançamento, recebimento de contagens de outro coletor, reimportação do mesmo arquivo sem duplicar, arquivo atualizado trazendo só o que faltava, soma correta de produtos contados em dois aparelhos, snapshot e restauração após um "zerar", e a ida e volta completa pela planilha (exporta, relê a aba LANÇAMENTOS e reimporta sem duplicar nem alterar quantidades).

*Lógica (27):* integridade do catálogo (360 produtos, sem duplicidade), busca de código em todas as variações, leitura de etiqueta de balança, gravação e soma de peças, compatibilidade com lançamentos antigos sem o campo de peças, acúmulo de contagens sem substituição, formatação numérica pt-BR, os dois formatos de importação e a geração real do .xlsx com as três abas.

*Percentuais e planilha (12):* totais por unidade, participação de cada item fechando exatamente 100% em KG e em UN separadamente, % dentro do departamento, garantia de que KG e UN não se misturam, média por pesagem, ordem cronológica das pesagens, e a geração real do .xlsx com as quatro abas, conferida célula a célula (valores, formatos numéricos, cores de cabeçalho e totais, filtros e painéis congelados).

*Interface, simulando teclado e leitor num navegador de verdade (33):* três pesagens seguidas no mesmo produto sem redigitar o código, limpeza dos campos e retorno do cursor, acumulado da sessão, troca de produto por leitura com o cursor no campo de peso (e a garantia de que os dígitos não sujam o peso), digitação humana de 11 dígitos que não é confundida com leitura, comportamento distinto de itens KG e UN, `Esc`, o modo "trocar sempre" e o cadastro de produto novo no meio da contagem.
