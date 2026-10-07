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


