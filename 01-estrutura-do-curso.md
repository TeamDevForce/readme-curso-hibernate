# Hibernate + JPA com Java e PostgreSQL

Curso introdutório de **Hibernate + JPA** utilizando **Java**, **Maven** e **PostgreSQL**.

Durante o curso, vamos construir o mesmo pequeno sistema de cadastro de clientes utilizado no curso de JDBC, agora utilizando **JPA como especificação de persistência** e **Hibernate como implementação ORM**.

A proposta é entender como podemos trabalhar com objetos Java e persistência sem precisar escrever manualmente todo o código JDBC utilizado anteriormente.

---

## 🎯 Objetivo do curso

Ao final do curso, o aluno deverá ser capaz de:

* compreender o que é ORM;
* compreender o que é JPA;
* compreender o que é Hibernate;
* entender a diferença entre especificação e implementação;
* configurar Hibernate e JPA em um projeto Maven;
* conectar uma aplicação Java ao PostgreSQL;
* criar entidades utilizando anotações JPA;
* utilizar `EntityManager`;
* utilizar `EntityManagerFactory`;
* compreender o contexto de persistência;
* trabalhar com transações;
* executar operações de CRUD;
* utilizar JPQL;
* aplicar o padrão DAO;
* utilizar interfaces para desacoplar implementações;
* criar uma camada de Service;
* separar persistência de regras de negócio;
* organizar uma aplicação Java utilizando Hibernate e JPA.

---

# 📚 Estrutura do curso

O conteúdo será dividido em quatro arquivos:

```text
01-estrutura-do-curso.md
02-setup-e-conexao.md
03-crud-dao-e-interface.md
04-service-e-regras-de-negocio.md
```

---

# Parte 1 — Setup e Conexão

## Objetivo

Entender os conceitos fundamentais de ORM, JPA e Hibernate e configurar nossa aplicação para se comunicar com o PostgreSQL.

## Conteúdo

* diferença entre JDBC, JPA e Hibernate;
* conceito de ORM;
* conceito de especificação;
* conceito de implementação;
* criação do projeto Maven;
* configuração do `pom.xml`;
* dependência do Hibernate;
* driver PostgreSQL;
* criação do banco;
* configuração do `persistence.xml`;
* conceito de unidade de persistência;
* conceito de `EntityManagerFactory`;
* conceito de `EntityManager`;
* criação da `JPAUtil`;
* teste da configuração.

## Fluxo

```text
Aplicação
    ↓
JPA
    ↓
Hibernate
    ↓
JDBC
    ↓
PostgreSQL
```

---

# Parte 2 — CRUD, DAO e Interface

## Objetivo

Criar uma entidade e realizar operações de persistência utilizando JPA e Hibernate.

## Conteúdo

* criação da entidade `Cliente`;
* `@Entity`;
* `@Table`;
* `@Id`;
* `@GeneratedValue`;
* `@Column`;
* mapeamento objeto-relacional;
* `EntityManager`;
* transações;
* `persist()`;
* `find()`;
* JPQL;
* `merge()`;
* `remove()`;
* criação da interface `ClienteDAO`;
* criação da implementação `ClienteDAOImpl`;
* separação das responsabilidades de persistência.

## CRUD desenvolvido

| Operação | JPA | Método |
| --- | --- | --- |
| Criar | `persist()` | `salvar()` |
| Consultar | `find()` | `buscarPorId()` |
| Listar | JPQL | `listarTodos()` |
| Atualizar | `merge()` | `atualizar()` |
| Excluir | `remove()` | `excluir()` |

---

# Parte 3 — Service e Regras de Negócio

## Objetivo

Separar regras de negócio da lógica responsável pela persistência.

## Conteúdo

* criação da `ClienteService`;
* diferença entre persistência e regra de negócio;
* validações;
* dependência através da interface `ClienteDAO`;
* injeção de dependência pelo construtor;
* comunicação entre Service e DAO;
* montagem das dependências na `Main`;
* fluxo completo da aplicação.

## Fluxo

```text
Main
  ↓
ClienteService
  ↓
ClienteDAO
  ↓
ClienteDAOImpl
  ↓
EntityManager
  ↓
Hibernate
  ↓
PostgreSQL
```

---

# 🗂️ Estrutura do projeto

```text
hibernate-jpa-clientes
│
├── pom.xml
│
└── src
    └── main
        │
        ├── java
        │   └── com
        │       └── devforce
        │           └── jpa
        │               │
        │               ├── Main.java
        │               │
        │               ├── model
        │               │   └── Cliente.java
        │               │
        │               ├── dao
        │               │   ├── ClienteDAO.java
        │               │   └── ClienteDAOImpl.java
        │               │
        │               ├── service
        │               │   └── ClienteService.java
        │               │
        │               └── config
        │                   └── JPAUtil.java
        │
        └── resources
            └── META-INF
                └── persistence.xml
```

---

# 🧩 Entidade utilizada

Continuaremos utilizando o exemplo de `Cliente`.

```text
Cliente

id
nome
dataNascimento
criadoEm
```

Agora, porém, a classe será uma entidade gerenciada pelo JPA.

---

# 🔄 Comparando JDBC e JPA

No JDBC:

```text
Cliente
   ↓
DAO
   ↓
Connection
   ↓
PreparedStatement
   ↓
SQL
   ↓
PostgreSQL
```

Com JPA e Hibernate:

```text
Cliente
   ↓
DAO
   ↓
EntityManager
   ↓
Hibernate
   ↓
PostgreSQL
```

O Hibernate continuará utilizando JDBC internamente.

A diferença é que nossa aplicação passa a trabalhar principalmente com objetos Java.

---

# 🏁 Resultado final

Ao final do curso teremos:

```text
Main
 │
 ▼
ClienteService
 │
 ▼
ClienteDAO
 │
 ▼
ClienteDAOImpl
 │
 ▼
EntityManager
 │
 ▼
JPA
 │
 ▼
Hibernate
 │
 ▼
JDBC
 │
 ▼
PostgreSQL
```

O objetivo não é esconder completamente o funcionamento do banco de dados.

A ideia é entender como **JPA e Hibernate abstraem parte do trabalho manual realizado anteriormente com JDBC**, permitindo que nossa aplicação trabalhe de forma mais orientada a objetos.
