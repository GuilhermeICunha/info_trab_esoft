# Roteiro de apresentação – até 2 minutos por parte

Apresentem na ordem 1 → 5. Cada fala tem cerca de 250 palavras (≈ 2 min). Quem fala deve abrir os arquivos citados e mostrar os blocos pelo ID.
Dica: ao terminar, cada pessoa "passa a bola" dizendo qual parte vem a seguir.

---
## PARTE 1 – Banco de dados e configuração
**Objetivo:** criar a base onde tudo é guardado e conectar o PHP a ela.

**Fala:**
"Nosso projeto é um sistema de livraria com duas entidades: **Categorias** e **Livros**, onde cada livro pertence a uma categoria. Eu apresento a base de tudo: o banco e a configuração.

Começando pelo `banco.sql`. No bloco **P1-01** criamos o banco `livraria_crud`. No **P1-02** criamos a tabela `categorias`, com nome único. No **P1-03** criamos a tabela `livros`, que tem a coluna `categoria_id`, uma **chave estrangeira** para categorias. Ela usa `ON DELETE RESTRICT`: o banco impede apagar uma categoria que ainda tenha livros. No **P1-04** inserimos dados de exemplo para testar.

Agora o `config/conexao.php`. No **P1-05** ficam as constantes: servidor, banco, usuário, senha e a pasta base do site. No **P1-06** a função `conectar()` abre a conexão com **PDO**, uma única vez, e configura os erros como exceção. Todo o resto do projeto chama essa função. No **P1-07** a função `e()` protege a tela contra injeção de HTML, e no **P1-08** a função `redirecionar()` leva o usuário a outra página levando uma mensagem de sucesso ou erro.

Resumindo: o banco guarda os dados, o `conexao.php` abre a porta de acesso e oferece ferramentas que as outras quatro partes vão usar. A Parte 2 mostra como as categorias usam essa conexão."

---
## PARTE 2 – Categorias: Model e Controller
**Objetivo:** executar no banco todas as operações de categorias e controlar as regras de negócio.

**Fala:**
"Eu apresento o lado das categorias, que não aparece na tela, mas faz o trabalho: o Model e o Controller.

O **Model**, `Categoria.php`, só conversa com o banco. No **P2-01** ele recebe a conexão. Depois temos uma função para cada operação do CRUD: **P2-02** `listar`, **P2-03** `buscar` um por id, **P2-04** `inserir`, **P2-05** `atualizar` e **P2-06** `excluir`. Todas usam *prepared statements*, que protegem contra SQL Injection.

O **Controller**, `CategoriaController.php`, recebe os formulários. No **P2-07** ele carrega os arquivos, lê a ação da URL e recusa qualquer acesso que não seja POST. No **P2-08**, a ação `salvar`: lê os campos, valida se o nome foi preenchido e decide: se veio um `id`, chama `atualizar`; se não, chama `inserir`. Também trata nome repetido, usando o erro 1062 do banco. No **P2-09**, a ação `excluir`: se a categoria tiver livros, o banco bloqueia com o erro 1451 e o usuário recebe uma mensagem clara. O **P2-10** manda qualquer outra coisa de volta à listagem.

Ou seja: a View, que a Parte 4 mostra, envia dados ao Controller; o Controller valida e chama o Model; o Model fala com o banco da Parte 1. Terminado, o controller redireciona de volta com uma mensagem."

---
## PARTE 3 – Livros: Model e Controller
**Objetivo:** executar as operações de livros, incluindo o relacionamento com categorias, e validar mais campos.

**Fala:**
"Eu apresento o mesmo caminho, agora para os livros, que são a entidade principal e dependem das categorias.

No **Model**, `Livro.php`, o **P3-01** recebe a conexão. O **P3-02**, `listar`, é o destaque: usa um **INNER JOIN** entre `livros` e `categorias`, para trazer o *nome* da categoria em vez de só o número. Depois vêm `buscar` (P3-03), `inserir` (P3-04), `atualizar` (P3-05) e `excluir` (P3-06), todos com prepared statements.

No **Controller**, `LivroController.php`, o **P3-07** prepara tudo e só aceita POST. O **P3-08**, `salvar`, valida mais campos que o de categorias: título e autor preenchidos, preço numérico e não negativo (aceitando vírgula, como 89,90), estoque inteiro e não negativo, e categoria escolhida. Se algo falhar, volta ao formulário com mensagem de erro. Passando, decide entre `inserir` e `atualizar` pelo `id`. O **P3-09** exclui o livro; livro não tem dependentes, então apagar é simples.

A ligação com o resto: o formulário da Parte 5 envia o `categoria_id` escolhido; o controller valida; o model grava na coluna de chave estrangeira criada no **P1-03**. É assim que o relacionamento entre as duas tabelas funciona na prática."

---
## PARTE 4 – Layout, CSS e telas de Categorias
**Objetivo:** dar a aparência comum ao sistema e criar as telas de categorias.

**Fala:**
"Eu apresento o que o usuário vê: o layout comum e as telas de categorias.

O `header.php`, no **P4-01**, abre a página, carrega o CSS e mostra o menu. No **P4-02** ele exibe as mensagens de sucesso ou erro que os controllers mandam pela URL. O `footer.php`, **P4-03**, fecha a página. Todas as telas incluem os dois, então o visual é padronizado. O **P4-04**, `estilo.css`, é o CSS externo com cores, tabelas, botões e alertas. O **P4-05**, `index.php`, é a página inicial com os atalhos.

Nas categorias, a `listar.php` tem dois blocos: no **P4-06** ela pede ao Model a lista de categorias, e no **P4-07** mostra a tabela com os botões *Editar* e *Excluir*. O Excluir é um pequeno formulário POST com confirmação em JavaScript, que chama o controller da Parte 2.

A `form.php` serve para cadastrar e editar. No **P4-08**, se a URL tem um `id`, ela busca a categoria no Model e preenche os campos; senão, abre vazia. No **P4-09** está o formulário, que envia por POST para o controller, ação `salvar`.

Então: a View nunca grava no banco. Ela lê pelo Model, e para gravar ou apagar sempre envia ao Controller."

---
## PARTE 5 – Telas de Livros e fluxo completo
**Objetivo:** criar as telas de livros, ligar o livro à categoria e demonstrar o sistema funcionando.

**Fala:**
"Eu apresento as telas de livros e fecho mostrando o sistema todo funcionando.

A `listar.php` de livros: no **P5-01** ela chama `Livro::listar`, que já traz o nome da categoria pelo JOIN da Parte 3. No **P5-02** mostra a tabela, formatando o preço em reais com `number_format` e usando `e()` para proteger o texto. Os botões Editar e Excluir seguem o mesmo padrão das categorias.

A `form.php` de livros tem um detalhe importante. No **P5-03**, além de buscar o livro para edição, ela chama `Categoria::listar` para obter as categorias. No **P5-04**, o formulário monta um **select** com essas categorias e marca a atual na edição. Se não houver categorias, o sistema pede para cadastrá-las antes. Esse campo envia o `categoria_id` ao controller, que o grava na chave estrangeira do banco.

Agora a demonstração: 1) abrimos Categorias e criamos uma; 2) abrimos Livros e criamos um livro nessa categoria; 3) editamos o preço; 4) tentamos excluir a categoria e vemos o bloqueio, porque ainda há livro nela; 5) excluímos o livro e depois a categoria. Esse caminho passa pelas cinco partes."
