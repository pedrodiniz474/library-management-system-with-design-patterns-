# 📚 BibliotecaWeb

Sistema de gerenciamento de biblioteca desenvolvido com Java EE, seguindo os padrões de projeto MVC, DAO, Command, Builder e Decorator.

---

## 🗂 Índice

- [Sobre o projeto](#-sobre-o-projeto)
- [Funcionalidades](#-funcionalidades)
- [Tecnologias](#-tecnologias)
- [Padrões de Projeto](#-padrões-de-projeto)
- [Arquitetura](#-arquitetura)
- [Estrutura de Pacotes](#-estrutura-de-pacotes)
- [Pré-requisitos](#-pré-requisitos)
- [Configuração do Banco de Dados](#-configuração-do-banco-de-dados)
- [Como Executar](#-como-executar)
- [Páginas da Aplicação](#-páginas-da-aplicação)
- [Diagramas UML](#-diagramas-uml)

---

## 📖 Sobre o projeto

O **BibliotecaWeb** é uma aplicação web Java EE para gerenciamento de acervo de biblioteca. Permite que administradores cadastrem livros, usuários e empréstimos, e que usuários comuns solicitem e devolvam livros diretamente pelo sistema. O controle de multas por atraso é feito automaticamente com o padrão Decorator.

---

## ✅ Funcionalidades

### Administrador
- Login com autenticação por e-mail e senha (SHA-256)
- Cadastrar, editar e excluir **livros**
- Cadastrar, editar e excluir **usuários**
- Cadastrar, editar, excluir e **registrar devolução** de empréstimos
- Visualizar estantes por gênero literário
- Dashboard de controle

### Usuário
- Auto-cadastro na plataforma
- Login com redirecionamento para área pessoal
- Solicitar empréstimo de livro (bloqueado automaticamente se houver multa pendente)
- Devolver livro com cálculo automático de multa por atraso
- Visualizar seus empréstimos e estantes

---

## 🛠 Tecnologias

| Tecnologia | Versão |
|---|---|
| Java | 8 |
| Java EE (javax) | 8.0 |
| Jakarta Servlets / JSP | 4.0 |
| MySQL Connector/J | 8.3.0 |
| Maven | 3.9.9 |
| Servidor de aplicação | Apache Tomcat 8.5.96 |
| Banco de dados | MySQL Workbench 8.0 CE |
| IDE | NetBeans 25 |

---

## 🧩 Padrões de Projeto

### MVC (Model-View-Controller)
A aplicação é dividida em três camadas bem definidas. A View é composta pelas páginas JSP. O Controller é formado pelos Servlets, que recebem as requisições e delegam a execução. O Model contém as entidades, enums e serviços de negócio.

### DAO (Data Access Object)
Cada entidade possui suas próprias classes DAO para isolar o acesso ao banco de dados. Todas implementam as interfaces `IDAO<T>` (para operações de escrita) ou `IConsultar<T>` (para leitura), garantindo um contrato uniforme.

### Command
O `ManterServlet` não executa nenhuma lógica diretamente. Ele delega para o `CommandFactory`, que instancia via Reflection a classe `Action` correta com base nos parâmetros `entidade` e `btnop` recebidos da requisição. Cada `Action` implementa a interface `ICommand`.

### Builder
As entidades `Livro` e `Emprestimo` utilizam inner classes `Builder` para construção fluente, evitando construtores com muitos parâmetros e garantindo legibilidade na criação de objetos.

### Decorator
O cálculo de multa por atraso usa o padrão Decorator. `MultaBase` calcula R$ 2,00 por dia de atraso. `MultaAtrasoExtendido` envolve a `MultaBase` e adiciona uma penalidade fixa de R$ 10,00 quando o atraso supera 7 dias. `MultaNotificacaoAtraso` pode ser composta para registrar notificações sem alterar o cálculo base.

---

## 🏛 Arquitetura

O fluxo de uma requisição segue o caminho abaixo:

```
JSP (View)
   ↓ HTTP POST/GET
Servlet (Controller)
   ↓ delega
CommandFactory → ICommand (Action)
   ↓ chama
DAO
   ↓ executa SQL
MySQL
```

O `ManterServlet` é o ponto central para todas as operações de manutenção (livros, usuários, empréstimos). `LoginServlet`, `CadastroServlet` e `LogoutServlet` são servlets independentes para autenticação.

---

## 📁 Estrutura de Pacotes

```
src/main/java/
├── controller/
│   ├── LoginServlet.java
│   ├── CadastroServlet.java
│   ├── ManterServlet.java
│   └── LogoutServlet.java
│
├── command/
│   ├── ICommand.java
│   ├── CommandFactory.java
│   ├── livro/
│   │   ├── CadastrarLivroAction.java
│   │   ├── AtualizarLivroAction.java
│   │   └── DeletarLivroAction.java
│   ├── usuario/
│   │   ├── CadastrarUsuarioAction.java
│   │   ├── AtualizarUsuarioAction.java
│   │   └── DeletarUsuarioAction.java
│   └── emprestimo/
│       ├── CadastrarEmprestimoAction.java
│       ├── AtualizarEmprestimoAction.java
│       ├── DeletarEmprestimoAction.java
│       └── DevolverEmprestimoAction.java
│
├── dao/
│   ├── IDAO.java
│   ├── IConsultar.java
│   ├── livro/
│   ├── usuario/
│   ├── emprestimo/
│   └── estante/
│
├── model/
│   ├── entidade/
│   │   ├── Livro.java          (+ LivroBuilder)
│   │   ├── Usuario.java
│   │   ├── Emprestimo.java     (+ EmprestimoBuilder)
│   │   ├── Estante.java        (abstract)
│   │   ├── EstanteFiccao.java
│   │   ├── EstanteRomance.java
│   │   ├── EstanteTerror.java
│   │   └── EstanteAventura.java
│   ├── enums/
│   │   ├── TipoUsuario.java    (ADMIN, USER)
│   │   └── StatusLivro.java    (DISPONIVEL, EMPRESTADO, INATIVO)
│   └── service/
│       ├── CalculadoraMulta.java
│       └── multa/
│           ├── ICalculoMulta.java
│           ├── MultaBase.java
│           ├── MultaDecorator.java
│           ├── MultaAtrasoExtendido.java
│           └── MultaNotificacaoAtraso.java
│
└── util/
    ├── ConnectionFactory.java
    └── HashUtil.java

src/main/webapp/
├── index.jsp
├── cadastro.jsp
├── dashboard.jsp
├── home.jsp
├── livros.jsp
├── usuarios.jsp
├── emprestimos.jsp
├── estantes.jsp
└── WEB-INF/
    └── web.xml
```

---

## ⚙️ Pré-requisitos

- Java JDK 8
- Apache Tomcat 8.5.96
- MySQL Workbench 8.0 CE
- Maven 3.9.9
- NetBeans 25

---

## 🗄 Configuração do Banco de Dados

**1. Crie o banco de dados no MySQL:**

```sql
CREATE DATABASE BibliotecaWeb;

USE BibliotecaWeb;
```

**2. Crie as tabelas:**

```sql
CREATE TABLE usuarios (
    id      INT AUTO_INCREMENT PRIMARY KEY,
    nome    VARCHAR(100) NOT NULL,
    email   VARCHAR(100) NOT NULL UNIQUE,
    senha   VARCHAR(64)  NOT NULL,
    tipo    ENUM('ADMIN', 'USER') NOT NULL DEFAULT 'USER',
    ativo   BOOLEAN NOT NULL DEFAULT TRUE
);

CREATE TABLE livros (
    id              INT AUTO_INCREMENT PRIMARY KEY,
    titulo          VARCHAR(200) NOT NULL,
    autor           VARCHAR(100) NOT NULL,
    editora         VARCHAR(100),
    isbn            VARCHAR(20),
    genero          VARCHAR(50),
    ano_publicacao  INT,
    numero_paginas  INT,
    idioma          VARCHAR(30),
    preco           DECIMAL(10,2),
    quantidade      INT NOT NULL DEFAULT 0,
    status          ENUM('DISPONIVEL', 'EMPRESTADO', 'INATIVO') NOT NULL DEFAULT 'DISPONIVEL'
);

CREATE TABLE estantes (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    genero VARCHAR(50) NOT NULL
);

CREATE TABLE emprestimos (
    id                      INT AUTO_INCREMENT PRIMARY KEY,
    id_usuario              INT NOT NULL,
    id_livro                INT NOT NULL,
    data_emprestimo         DATE NOT NULL,
    data_prevista_devolucao DATE NOT NULL,
    data_devolucao          DATE,
    FOREIGN KEY (id_usuario) REFERENCES usuarios(id),
    FOREIGN KEY (id_livro)   REFERENCES livros(id)
);
```

**3. Insira um administrador inicial:**

```sql
-- Senha: admin123 (SHA-256)
INSERT INTO usuarios (nome, email, senha, tipo, ativo)
VALUES (
    'Administrador',
    'admin@biblioteca.com',
    '240be518fabd2724ddb6f04eeb1da5967448d7e831c08c8fa822809f74c720a9',
    'ADMIN',
    true
);
```

**4. Insira as estantes padrão:**

```sql
INSERT INTO estantes (genero) VALUES ('Ficcao'), ('Romance'), ('Terror'), ('Aventura');
```

**5. Configure a conexão** em `src/main/resources/db.properties`:

```properties
db.url=jdbc:mysql://localhost:3306/BibliotecaWeb?useUnicode=true&characterEncoding=UTF-8
db.user=root
db.password=sua_senha_aqui
db.driver=com.mysql.cj.jdbc.Driver
```

---

## ▶️ Como Executar

**1. Clone o repositório:**
```bash
git clone https://github.com/seu-usuario/BibliotecaWeb.git
cd BibliotecaWeb
```

**2. Configure o banco** conforme a seção acima.

**3. Build com Maven:**
```bash
mvn clean package
```

**4. Deploy no Tomcat:**
- Copie o `.war` gerado em `target/BibliotecaWeb-1.0-SNAPSHOT.war` para a pasta `webapps` do Tomcat, ou
- No NetBeans/IntelliJ, configure o Tomcat como servidor e rode direto pela IDE.

**5. Acesse no navegador:**
```
http://localhost:8080/BibliotecaWeb
```

---

## 🖥 Páginas da Aplicação

| Página | Acesso | Descrição |
|---|---|---|
| `index.jsp` | Público | Tela de login |
| `cadastro.jsp` | Público | Auto-cadastro de novo usuário |
| `dashboard.jsp` | Admin | Painel principal do administrador |
| `home.jsp` | Usuário | Área do usuário: livros e empréstimos |
| `livros.jsp` | Admin | CRUD completo de livros |
| `usuarios.jsp` | Admin | CRUD completo de usuários |
| `emprestimos.jsp` | Admin | Gerenciamento de empréstimos e devoluções |
| `estantes.jsp` | Ambos | Visualização de livros por gênero |

---

## 📐 Diagramas UML

Os diagramas foram gerados com PlantUML e estão disponíveis na pasta `/diagramas` do repositório.

### Diagrama de Classes

Organizado em camadas da esquerda para a direita: `util → enums → service → entidade → dao → command → controller → webapp`.

Relacionamentos presentes:
- **Herança** — subclasses de `Estante` e de `MultaDecorator`
- **Implementação** — `ICommand`, `IDAO<T>`, `IConsultar<T>`, `ICalculoMulta`
- **Associação 1:N** — `Usuario` possui vários `Emprestimo`; `Livro` aparece em vários `Emprestimo`
- **Associação 1:1** — cada `Livro` pertence a uma `Estante`
- **Agregação** — `Estante` agrega `Livro`
- **Composição** — inner classes `LivroBuilder` e `EmprestimoBuilder`
- **Dependência** — camadas superiores dependem das inferiores sem referência permanente

### Diagrama de Sequência — Administrador

Cobre os fluxos: login, CRUD de livros, CRUD de usuários, cadastro/atualização/devolução/exclusão de empréstimos e logout.

### Diagrama de Sequência — Usuário

Cobre os fluxos: auto-cadastro, login, solicitação de empréstimo (com bloqueio por multa pendente), devolução de livro (com cálculo de multa) e logout.

---

## 👨‍💻 Autor

**Pedro Henrique Diniz Matos**  
Universidade de Mogi das Cruzes — Engenharia de Software
