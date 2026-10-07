# Parte 01 — Setup e Conexão

## Objetivo

Entender o que são **ORM**, **JPA** e **Hibernate**, configurar o projeto e estabelecer a comunicação entre a aplicação Java e o PostgreSQL.

---

# 1. O que é ORM?

ORM significa:

**Object-Relational Mapping**

ou:

**Mapeamento Objeto-Relacional**

A ideia é criar uma relação entre:

```text
Objeto Java
     ↕
Tabela do banco
```

Por exemplo:

```text
Cliente.java
     ↕
tabela cliente
```

E também:

```text
Java                PostgreSQL

id                  id
nome                nome
dataNascimento      data_nascimento
criadoEm            criado_em
```

---

# 2. O que é JPA?

JPA significa:

**Jakarta Persistence API**

JPA é uma **especificação**.

Ela define regras, interfaces e anotações para trabalhar com persistência em aplicações Java.

Por exemplo:

```java
@Entity
```

```java
@Id
```

```java
@Column
```

```java
EntityManager
```

A JPA define como essas ferramentas devem funcionar.

Mas ela não realiza todo o trabalho sozinha.

---

# 3. O que é Hibernate?

Hibernate é uma implementação da especificação JPA.

Podemos representar assim:

```text
JPA
↓
Define as regras

Hibernate
↓
Implementa essas regras
```

Ou:

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

# 4. Criando o projeto Maven

Crie um novo projeto Maven.

Exemplo:

```text
groupId:
com.devforce

artifactId:
hibernate-jpa-clientes
```

---

# 5. Configurando o pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.devforce</groupId>
    <artifactId>hibernate-jpa-clientes</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>26</maven.compiler.source>
        <maven.compiler.target>26</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>7.4.12.Final</version>
        </dependency>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.7.7</version>
        </dependency>

    </dependencies>

</project>
```

---

# 6. Criando o banco

No PostgreSQL:

```sql
CREATE DATABASE devforce_jpa;
```

---

# 7. Criando o persistence.xml

Crie:

```text
src/main/resources/META-INF/persistence.xml
```

Exemplo:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<persistence
        xmlns="https://jakarta.ee/xml/ns/persistence"
        version="3.2">

    <persistence-unit name="devforce-jpa">

        <provider>
            org.hibernate.jpa.HibernatePersistenceProvider
        </provider>

        <class>
            com.devforce.jpa.model.Cliente
        </class>

        <properties>

            <property
                    name="jakarta.persistence.jdbc.url"
                    value="jdbc:postgresql://localhost:5432/devforce_jpa"/>

            <property
                    name="jakarta.persistence.jdbc.user"
                    value="postgres"/>

            <property
                    name="jakarta.persistence.jdbc.password"
                    value="postgres"/>

            <property
                    name="jakarta.persistence.jdbc.driver"
                    value="org.postgresql.Driver"/>

            <property
                    name="hibernate.hbm2ddl.auto"
                    value="update"/>

            <property
                    name="hibernate.show_sql"
                    value="true"/>

            <property
                    name="hibernate.format_sql"
                    value="true"/>

        </properties>

    </persistence-unit>

</persistence>
```

---

# 8. Unidade de persistência

No arquivo temos:

```xml
<persistence-unit name="devforce-jpa">
```

Esse nome identifica nossa unidade de persistência.

Depois utilizaremos:

```java
Persistence.createEntityManagerFactory(
        "devforce-jpa"
);
```

---

# 9. EntityManagerFactory

O `EntityManagerFactory` é responsável por criar `EntityManager`.

```text
EntityManagerFactory
        ↓
EntityManager
```

Criar um `EntityManagerFactory` é uma operação relativamente cara.

Por isso, normalmente mantemos apenas uma instância durante a execução da aplicação.

---

# 10. EntityManager

O `EntityManager` é um dos principais componentes da JPA.

Ele é responsável por operações como:

```text
persistir
buscar
atualizar
remover
consultar
```

Exemplos:

```java
entityManager.persist(cliente);
```

```java
entityManager.find(
        Cliente.class,
        1
);
```

---

# 11. Criando a JPAUtil

Vamos centralizar a criação do `EntityManager`.

```java
package com.devforce.jpa.config;

import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.Persistence;

public class JPAUtil {

    private static final EntityManagerFactory FACTORY =
            Persistence.createEntityManagerFactory(
                    "devforce-jpa"
            );

    private JPAUtil() {
    }

    public static EntityManager getEntityManager() {

        return FACTORY.createEntityManager();
    }

    public static void close() {

        FACTORY.close();
    }
}
```

Agora podemos solicitar:

```java
EntityManager entityManager =
        JPAUtil.getEntityManager();
```

---

# 12. Fluxo

```text
JPAUtil
   ↓
EntityManagerFactory
   ↓
EntityManager
   ↓
Hibernate
   ↓
JDBC
   ↓
PostgreSQL
```

---

# EntityManagerFactory != EntityManager

É importante entender a diferença.

O `EntityManagerFactory` normalmente é reutilizado:

```text
EntityManagerFactory
        ↓
        ├── EntityManager 1
        ├── EntityManager 2
        └── EntityManager 3
```

Não devemos utilizar um único `EntityManager` aberto indefinidamente para toda a aplicação.

---

# Resumo

Nesta etapa aprendemos:

* ORM;
* JPA;
* Hibernate;
* diferença entre especificação e implementação;
* Maven;
* configuração do Hibernate;
* `persistence.xml`;
* unidade de persistência;
* `EntityManagerFactory`;
* `EntityManager`;
* `JPAUtil`;
* comunicação com PostgreSQL.

Na próxima etapa vamos transformar nossa classe `Cliente` em uma entidade JPA e implementar o CRUD.
