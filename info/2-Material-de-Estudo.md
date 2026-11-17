# Material de estudo – Livraria CRUD

## Visão geral (leiam antes das partes)
**O que o sistema faz:** cadastra **Categorias** e **Livros**. Cada livro pertence a uma categoria (relacionamento 1 → N).
**CRUD:** Create (criar), Read (ler), Update (editar), Delete (excluir) – existe para as duas entidades.

**Arquitetura (MVC simples):**
| Camada | Pasta | Responsabilidade |
|---|---|---|
| Configuração | `config/` | conexão com o banco e funções auxiliares |
| Model | `models/` | único lugar que escreve SQL |
| Controller | `controllers/` | recebe formulários, valida, chama o Model, redireciona |
| View | `views/` | HTML que o usuário vê |
| CSS | `css/` | aparência |

**Regra de ouro:** a View lê dados pelo Model; para gravar ou apagar, a View envia um POST ao Controller.
**Identificadores:** `[Px-yy]` = parte x, bloco yy. Procurem esses IDs nos comentários do código.

**Caminho de um cadastro:** formulário (View) → POST → Controller (valida) → Model (INSERT) → MySQL → redireciona à listagem (View) com mensagem.

---
## PARTE 1 – Banco de dados e configuração
**Arquivos:** `banco.sql`, `config/conexao.php`
**Objetivo:** guardar os dados e dar ao PHP acesso a eles.

| Bloco | O que é |
|---|---|
| P1-01 | Cria e seleciona o banco `livraria_crud` |
| P1-02 | Tabela `categorias` (`id`, `nome` único, `descricao`) |
| P1-03 | Tabela `livros` (`id`, `titulo`, `autor`, `preco`, `estoque`, `categoria_id`) |
| P1-04 | Dados de exemplo |
| P1-05 | Constantes: host, banco, usuário, senha, `BASE_URL` |
| P1-06 | `conectar()`: abre a conexão PDO |
| P1-07 | `e()`: escapa texto para o HTML |
| P1-08 | `redirecionar()`: muda de página com mensagem |

**Conceitos para entender**
- **Chave primária (PK):** `id` com `AUTO_INCREMENT`; identifica cada linha.
- **Chave estrangeira (FK):** `livros.categoria_id` aponta para `categorias.id`. Garante que todo livro tenha categoria existente.
- **ON DELETE RESTRICT:** impede apagar categoria com livros. Esse bloqueio gera o erro 1451, tratado em P2-09.
- **UNIQUE no nome:** impede categorias repetidas (erro 1062, tratado em P2-08).
- **PDO:** biblioteca do PHP para falar com o MySQL. Em `conectar()`, `ERRMODE_EXCEPTION` faz os erros virarem exceções (`try/catch`), e `FETCH_ASSOC` devolve cada linha como array (`$linha['nome']`).
- **`static $pdo`:** a conexão é criada só na primeira chamada e reaproveitada depois.
- **`e()`:** usa `htmlspecialchars`; se alguém cadastrar `<script>`, aparece como texto e não executa.
- **`BASE_URL`:** pasta do site no servidor (`/crud_livraria/`). Se a pasta mudar de nome, ajuste aqui.

**Como se conecta com o resto:** P1-06 é chamada pelos controllers e views; P1-07 e P1-08 são usadas em quase todo arquivo.

---
## PARTE 2 – Categorias: Model e Controller
**Arquivos:** `models/Categoria.php`, `controllers/CategoriaController.php`
**Objetivo:** executar e controlar as operações de categorias.

| Bloco | O que faz | CRUD |
|---|---|---|
| P2-01 | Classe `Categoria`; recebe o PDO no construtor | – |
| P2-02 | `listar()`: todas as categorias | Read |
| P2-03 | `buscar($id)`: uma categoria | Read |
| P2-04 | `inserir()` | Create |
| P2-05 | `atualizar()` | Update |
| P2-06 | `excluir()` | Delete |
| P2-07 | Prepara o controller: includes, `$acao`, só aceita POST | – |
| P2-08 | Ação `salvar`: valida e escolhe inserir ou atualizar | C/U |
| P2-09 | Ação `excluir`: trata o erro de FK | D |
| P2-10 | Ação desconhecida volta à listagem | – |

**Conceitos para entender**
- **Prepared statement:** `prepare('... WHERE id = ?')` e `execute([$id])`. O valor vai separado do SQL, o que evita SQL Injection.
- **Como o controller sabe o que fazer:** pela URL `?acao=salvar` ou `?acao=excluir`.
- **Criar ou editar?** O formulário envia um campo oculto `id`. `id = 0` → inserir; `id > 0` → atualizar.
- **`trim()`:** remove espaços das pontas; `(int)` converte para número inteiro.
- **`$e->errorInfo[1]`:** código de erro do MySQL (1062 = duplicado, 1451 = FK impede exclusão).
- **POST apenas:** impede apagar algo só por abrir um link.

**Como se conecta com o resto:** usa `conectar()` (P1-06) e `redirecionar()` (P1-08); recebe dados de P4-09 e P4-07; devolve o usuário a P4-07 com mensagem exibida em P4-02.

---
## PARTE 3 – Livros: Model e Controller
**Arquivos:** `models/Livro.php`, `controllers/LivroController.php`
**Objetivo:** executar e controlar as operações de livros, ligadas à categoria.

| Bloco | O que faz | CRUD |
|---|---|---|
| P3-01 | Classe `Livro`; recebe o PDO | – |
| P3-02 | `listar()` com INNER JOIN (traz o nome da categoria) | Read |
| P3-03 | `buscar($id)` | Read |
| P3-04 | `inserir()` | Create |
| P3-05 | `atualizar()` | Update |
| P3-06 | `excluir()` | Delete |
| P3-07 | Prepara o controller | – |
| P3-08 | Ação `salvar` com validações | C/U |
| P3-09 | Ação `excluir` | D |
| P3-10 | Ação desconhecida volta à listagem | – |

**Conceitos para entender**
- **INNER JOIN:** `JOIN categorias c ON c.id = l.categoria_id` junta as tabelas e permite mostrar `c.nome AS categoria` ao lado de cada livro.
- **Validações em P3-08:** título e autor não vazios; preço numérico e ≥ 0; estoque inteiro e ≥ 0; categoria selecionada.
- **Vírgula no preço:** `str_replace(',', '.', ...)` transforma "89,90" em "89.90", formato que o PHP e o MySQL entendem.
- **`filter_var(..., FILTER_VALIDATE_INT)`:** confere se o estoque é inteiro.
- **Casts `(float)` e `(int)`:** garantem o tipo certo antes de ir ao banco.
- **Sem dependentes:** excluir livro não gera conflito de FK, por isso o P3-09 é mais simples que o P2-09.

**Como se conecta com o resto:** recebe `categoria_id` do select em P5-04; grava na FK de P1-03; alimenta a listagem P5-02.

---
## PARTE 4 – Layout, CSS e telas de Categorias
**Arquivos:** `views/layout/header.php`, `views/layout/footer.php`, `css/estilo.css`, `index.php`, `views/categorias/listar.php`, `views/categorias/form.php`
**Objetivo:** interface padronizada e telas de categorias.

| Bloco | O que faz |
|---|---|
| P4-01 | Abre o HTML, carrega o CSS, mostra o menu |
| P4-02 | Mostra a mensagem `?msg=` (verde = ok, vermelho = erro) |
| P4-03 | Fecha o HTML (rodapé) |
| P4-04 | CSS externo |
| P4-05 | Página inicial com atalhos |
| P4-06 | `categorias/listar.php`: busca as categorias no Model |
| P4-07 | Tabela com Editar e Excluir |
| P4-08 | `categorias/form.php`: modo novo ou edição |
| P4-09 | Formulário que envia ao controller |

**Conceitos para entender**
- **Layout reaproveitado:** cada tela faz `require header.php` no começo e `require footer.php` no fim. A variável `$titulo` é definida antes do include e usada no `<title>`.
- **Alternância de PHP e HTML:** `<?php foreach (...): ?> ... <?php endforeach; ?>` repete linhas da tabela; `<?= e($valor) ?>` imprime com segurança.
- **Botão Excluir:** é um `<form method="post">` com campo oculto `id` e `onsubmit="return confirm(...)"` para pedir confirmação.
- **Editar:** é só um link `form.php?id=3`. O P4-08 lê `$_GET['id']`, busca no Model e preenche os campos.
- **Mensagens:** o controller chama `redirecionar(url, texto, tipo)`; o header lê `$_GET['msg']`.
- **Segurança:** valores numéricos vão com `(int)` e textos com `e()`.

**Como se conecta com o resto:** usa `Categoria::listar/buscar` (P2-02/P2-03); envia POST ao P2-08 e P2-09; usa CSS de P4-04.

---
## PARTE 5 – Telas de Livros e fluxo completo
**Arquivos:** `views/livros/listar.php`, `views/livros/form.php`
**Objetivo:** telas de livros e ligação livro → categoria.

| Bloco | O que faz |
|---|---|
| P5-01 | `livros/listar.php`: busca os livros (com JOIN) |
| P5-02 | Tabela: título, autor, categoria, preço, estoque, ações |
| P5-03 | `livros/form.php`: carrega categorias e (se edição) o livro |
| P5-04 | Formulário com `<select>` de categorias |

**Conceitos para entender**
- **Preço em reais:** `number_format($preco, 2, ',', '.')` exibe 1234.5 como 1.234,50.
- **Select dinâmico:** um `foreach` sobre as categorias gera as `<option>`. O `value` é o `id` da categoria e o texto é o nome.
- **Marcar a categoria atual:** `selected` aparece quando `categoria_id` do livro = `id` da opção.
- **Proteção de fluxo:** sem categorias cadastradas, o formulário avisa e não deixa cadastrar livro.
- **Preço na edição:** é formatado com vírgula, e o controller (P3-08) converte de volta.

**Como se conecta com o resto:** usa `Livro::listar/buscar` (P3-02/P3-03) e `Categoria::listar` (P2-02); envia POST ao P3-08 e P3-09.

---
## Teste rápido do sistema (para treinar a demonstração)
1. Criar categoria "Teste". 2. Criar livro "Livro 1" nela. 3. Editar o preço para 50,5. 4. Tentar excluir "Teste" → mensagem de bloqueio (P2-09 + P1-03). 5. Excluir o livro e depois a categoria. 6. Tentar criar categoria com nome repetido → mensagem de erro (P2-08).

## Perguntas que o professor pode fazer
- *Por que separar Model, Controller e View?* Cada arquivo tem uma responsabilidade; fica mais fácil de entender e mudar.
- *Como evitam SQL Injection?* Prepared statements (P2-02 a P2-06, P3-02 a P3-06).
- *Como evitam XSS?* Função `e()` (P1-07).
- *O que acontece ao apagar categoria com livros?* O banco bloqueia (FK RESTRICT, P1-03) e o controller mostra mensagem (P2-09).
- *Como o sistema sabe se é cadastro ou edição?* Pelo campo oculto `id` (P4-09 e P5-04), lido em P2-08 e P3-08.
