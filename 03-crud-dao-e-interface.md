# `03-crud-dao-e-interface.md`

# Parte 02 — CRUD, DAO e Interface

## Objetivo

Criar a entidade `Cliente` e implementar o CRUD utilizando JPA e Hibernate.

Também vamos manter a organização utilizada no curso de JDBC através do padrão DAO.

---

# 1. Criando a entidade Cliente

```java
package com.devforce.jpa.model;

import jakarta.persistence.*;

import java.time.LocalDate;
import java.time.LocalDateTime;

@Entity
@Table(name = "cliente")
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(
            nullable = false,
            length = 100
    )
    private String nome;

    @Column(
            name = "data_nascimento",
            nullable = false
    )
    private LocalDate dataNascimento;

    @Column(name = "criado_em")
    private LocalDateTime criadoEm;

    public Cliente() {
    }

    public Cliente(
            String nome,
            LocalDate dataNascimento
    ) {
        this.nome = nome;
        this.dataNascimento = dataNascimento;
        this.criadoEm = LocalDateTime.now();
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }

    public LocalDate getDataNascimento() {
        return dataNascimento;
    }

    public void setDataNascimento(
            LocalDate dataNascimento
    ) {
        this.dataNascimento = dataNascimento;
    }

    public LocalDateTime getCriadoEm() {
        return criadoEm;
    }

    public void setCriadoEm(
            LocalDateTime criadoEm
    ) {
        this.criadoEm = criadoEm;
    }
}
```

---

# Entendendo as anotações

## @Entity

```java
@Entity
```

Informa ao JPA que essa classe representa uma entidade persistente.

---

## @Table

```java
@Table(name = "cliente")
```

Define qual tabela será utilizada.

---

## @Id

```java
@Id
```

Define a chave primária.

---

## @GeneratedValue

```java
@GeneratedValue(
        strategy = GenerationType.IDENTITY
)
```

Informa que o identificador será gerado pelo banco.

---

## @Column

```java
@Column(name = "data_nascimento")
```

Permite configurar a relação entre atributo Java e coluna da tabela.

---

# 2. Criando ClienteDAO

```java
package com.devforce.jpa.dao;

import com.devforce.jpa.model.Cliente;

import java.util.List;

public interface ClienteDAO {

    void salvar(Cliente cliente);

    Cliente buscarPorId(Integer id);

    List<Cliente> listarTodos();

    Cliente atualizar(Cliente cliente);

    void excluir(Integer id);
}
```

---

# 3. ClienteDAOImpl

```java
package com.devforce.jpa.dao;

import com.devforce.jpa.config.JPAUtil;
import com.devforce.jpa.model.Cliente;

import jakarta.persistence.EntityManager;

import java.util.List;

public class ClienteDAOImpl
        implements ClienteDAO {
}
```

---

# 4. Salvar

```java
@Override
public void salvar(Cliente cliente) {

    EntityManager entityManager =
            JPAUtil.getEntityManager();

    try {

        entityManager
                .getTransaction()
                .begin();

        entityManager.persist(cliente);

        entityManager
                .getTransaction()
                .commit();

    } catch (Exception e) {

        if (
                entityManager
                        .getTransaction()
                        .isActive()
        ) {

            entityManager
                    .getTransaction()
                    .rollback();
        }

        throw e;

    } finally {

        entityManager.close();
    }
}
```

---

# Entendendo persist()

```java
entityManager.persist(cliente);
```

O `persist()` informa ao JPA que queremos tornar aquele objeto persistente.

```text
Cliente
   ↓
persist()
   ↓
Hibernate
   ↓
INSERT
   ↓
PostgreSQL
```

Não precisamos escrever:

```sql
INSERT INTO cliente ...
```

manualmente.

---

# Transação

Antes de alterar dados:

```java
entityManager
        .getTransaction()
        .begin();
```

Depois:

```java
entityManager
        .getTransaction()
        .commit();
```

Se ocorrer um erro:

```java
entityManager
        .getTransaction()
        .rollback();
```

Fluxo:

```text
begin
 ↓
operação
 ↓
commit
```

Em caso de erro:

```text
begin
 ↓
erro
 ↓
rollback
```

---

# 5. Buscar por ID

```java
@Override
public Cliente buscarPorId(Integer id) {

    EntityManager entityManager =
            JPAUtil.getEntityManager();

    try {

        return entityManager.find(
                Cliente.class,
                id
        );

    } finally {

        entityManager.close();
    }
}
```

O método:

```java
find()
```

busca uma entidade utilizando sua chave primária.

```text
Cliente.class
+
id
↓
find()
↓
Cliente
```

---

# 6. Listar todos

Para listar todos utilizaremos JPQL.

```java
@Override
public List<Cliente> listarTodos() {

    EntityManager entityManager =
            JPAUtil.getEntityManager();

    try {

        return entityManager
                .createQuery(
                        """
                        SELECT c
                        FROM Cliente c
                        ORDER BY c.id
                        """,
                        Cliente.class
                )
                .getResultList();

    } finally {

        entityManager.close();
    }
}
```

---

# JPQL

JPQL significa:

**Jakarta Persistence Query Language**

Observe:

```text
SQL

SELECT *
FROM cliente
```

Enquanto na JPQL:

```text
SELECT c
FROM Cliente c
```

A JPQL trabalha com:

```text
Entidades
Atributos
Objetos
```

e não diretamente com nomes de tabelas e colunas.

---

# 7. Atualizar

```java
@Override
public Cliente atualizar(Cliente cliente) {

    EntityManager entityManager =
            JPAUtil.getEntityManager();

    try {

        entityManager
                .getTransaction()
                .begin();

        Cliente clienteAtualizado =
                entityManager.merge(cliente);

        entityManager
                .getTransaction()
                .commit();

        return clienteAtualizado;

    } catch (Exception e) {

        if (
                entityManager
                        .getTransaction()
                        .isActive()
        ) {

            entityManager
                    .getTransaction()
                    .rollback();
        }

        throw e;

    } finally {

        entityManager.close();
    }
}
```

O método:

```java
merge()
```

sincroniza os dados do objeto informado com o contexto de persistência.

---

# 8. Excluir

```java
@Override
public void excluir(Integer id) {

    EntityManager entityManager =
            JPAUtil.getEntityManager();

    try {

        entityManager
                .getTransaction()
                .begin();

        Cliente cliente =
                entityManager.find(
                        Cliente.class,
                        id
                );

        if (cliente != null) {

            entityManager.remove(cliente);
        }

        entityManager
                .getTransaction()
                .commit();

    } catch (Exception e) {

        if (
                entityManager
                        .getTransaction()
                        .isActive()
        ) {

            entityManager
                    .getTransaction()
                    .rollback();
        }

        throw e;

    } finally {

        entityManager.close();
    }
}
```

Para utilizar:

```java
remove()
```

o objeto precisa estar associado ao contexto de persistência.

Por isso primeiro buscamos:

```java
Cliente cliente =
        entityManager.find(
                Cliente.class,
                id
        );
```

e depois:

```java
entityManager.remove(cliente);
```

---

# CRUD completo

```text
CREATE
↓
persist()

READ
↓
find()
JPQL

UPDATE
↓
merge()

DELETE
↓
remove()
```

---

# Comparando com JDBC

## JDBC

```text
Connection
↓
PreparedStatement
↓
SQL
↓
ResultSet
```

## JPA

```text
EntityManager
↓
Entidade
↓
Hibernate
```

---

# Fluxo da aplicação até aqui

```text
Cliente
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

# Resumo

Nesta etapa aprendemos:

* entidades;
* `@Entity`;
* `@Table`;
* `@Id`;
* `@GeneratedValue`;
* `@Column`;
* `EntityManager`;
* transações;
* `persist()`;
* `find()`;
* JPQL;
* `merge()`;
* `remove()`;
* DAO;
* interface;
* implementação DAO.

---

---

# `04-service-e-regras-de-negocio.md`

# Parte 03 — Service e Regras de Negócio

## Objetivo

Criar uma camada responsável pelas regras de negócio da aplicação e separar essa responsabilidade da camada responsável pela persistência.

---

# Por que criar uma Service?

Nosso DAO possui uma responsabilidade:

```text
Persistência
```

Ele deve cuidar de operações relacionadas ao armazenamento e recuperação dos dados.

Por exemplo:

```text
persist()
find()
merge()
remove()
JPQL
```

Regras da aplicação não devem ficar no DAO.

Para isso utilizaremos uma Service.

---

# Separação de responsabilidades

```text
ClienteService
↓
Regra de negócio

ClienteDAO
↓
Definição das operações

ClienteDAOImpl
↓
Persistência

Hibernate
↓
ORM
```

---

# Criando ClienteService

```java
package com.devforce.jpa.service;

import com.devforce.jpa.dao.ClienteDAO;
import com.devforce.jpa.model.Cliente;

public class ClienteService {

    private final ClienteDAO clienteDAO;

    public ClienteService(
            ClienteDAO clienteDAO
    ) {

        this.clienteDAO =
                clienteDAO;
    }

    public void cadastrar(
            Cliente cliente
    ) {

        if (
                cliente.getNome() == null ||
                cliente.getNome().isBlank()
        ) {

            throw new IllegalArgumentException(
                    "Nome é obrigatório"
            );
        }

        clienteDAO.salvar(cliente);
    }
}
```

---

# Regra de negócio

Nesse exemplo:

```java
if (
        cliente.getNome() == null ||
        cliente.getNome().isBlank()
) {

    throw new IllegalArgumentException(
            "Nome é obrigatório"
    );
}
```

Essa regra pertence à aplicação.

Ela não está relacionada ao Hibernate nem ao banco.

Por isso fica na Service.

---

# Injeção de dependência

Observe:

```java
private final ClienteDAO clienteDAO;
```

e:

```java
public ClienteService(
        ClienteDAO clienteDAO
) {

    this.clienteDAO =
            clienteDAO;
}
```

A Service recebe sua dependência pelo construtor.

Ela depende da interface:

```text
ClienteDAO
```

e não diretamente:

```text
ClienteDAOImpl
```

---

# Fluxo

```text
ClienteService
      ↓
ClienteDAO
      ↓
ClienteDAOImpl
      ↓
EntityManager
      ↓
Hibernate
```

---

# Main

```java
package com.devforce.jpa;

import com.devforce.jpa.dao.ClienteDAO;
import com.devforce.jpa.dao.ClienteDAOImpl;
import com.devforce.jpa.model.Cliente;
import com.devforce.jpa.service.ClienteService;

import java.time.LocalDate;

public class Main {

    public static void main(String[] args) {

        ClienteDAO clienteDAO =
                new ClienteDAOImpl();

        ClienteService clienteService =
                new ClienteService(
                        clienteDAO
                );

        Cliente cliente =
                new Cliente(
                        "Marcelo",
                        LocalDate.of(
                                1992,
                                5,
                                20
                        )
                );

        clienteService.cadastrar(
                cliente
        );
    }
}
```

---

# Entendendo a Main

Criamos primeiro o DAO:

```java
ClienteDAO clienteDAO =
        new ClienteDAOImpl();
```

Depois fornecemos o DAO para a Service:

```java
ClienteService clienteService =
        new ClienteService(
                clienteDAO
        );
```

Depois criamos o objeto:

```java
Cliente cliente =
        new Cliente(
                "Marcelo",
                LocalDate.of(
                        1992,
                        5,
                        20
                )
        );
```

E chamamos:

```java
clienteService.cadastrar(
        cliente
);
```

---

# Fluxo completo

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
JPA
  ↓
Hibernate
  ↓
JDBC
  ↓
PostgreSQL
```

---

# Estrutura final

```text
com.devforce.jpa

├── Main.java
│
├── model
│   └── Cliente.java
│
├── dao
│   ├── ClienteDAO.java
│   └── ClienteDAOImpl.java
│
├── service
│   └── ClienteService.java
│
└── config
    └── JPAUtil.java
```

---

# Responsabilidades

```text
model
↓
Representação das entidades

dao
↓
Persistência

service
↓
Regras de negócio

config
↓
Configuração do JPA
```

---

# Atividade prática

O exemplo demonstrou o cadastro utilizando:

```text
Main
↓
Service
↓
DAO
↓
Hibernate
↓
Banco
```

Agora implemente na `ClienteService` as demais operações:

* buscar cliente por ID;
* listar todos;
* atualizar cliente;
* excluir cliente.

Depois teste cada operação na classe `Main`.

---

# Desafio

Crie outro pequeno CRUD utilizando a mesma estrutura.

A entidade é livre.

Exemplos:

```text
Produto
Livro
Funcionário
Aluno
Curso
Veículo
Pedido
Filme
```

Procure manter:

```text
model
dao
service
config
```

---

# Comparação final

## JDBC

```text
Java
↓
Service
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

## Hibernate + JPA

```text
Java
↓
Service
↓
DAO
↓
EntityManager
↓
Hibernate
↓
PostgreSQL
```

---

# Resumo

Neste curso aprendemos:

* ORM;
* JPA;
* Hibernate;
* entidades;
* mapeamento objeto-relacional;
* `EntityManagerFactory`;
* `EntityManager`;
* transações;
* `persist()`;
* `find()`;
* JPQL;
* `merge()`;
* `remove()`;
* DAO;
* interfaces;
* Service;
* regras de negócio;
* injeção de dependência;
* separação de responsabilidades.

O objetivo foi reconstruir o projeto desenvolvido anteriormente com JDBC utilizando uma camada de abstração maior.

Agora conseguimos enxergar claramente a evolução:

```text
JDBC
↓
JPA
↓
Hibernate
```

e entender o que frameworks como Hibernate fazem por nós.
