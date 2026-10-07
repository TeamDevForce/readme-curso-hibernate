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
