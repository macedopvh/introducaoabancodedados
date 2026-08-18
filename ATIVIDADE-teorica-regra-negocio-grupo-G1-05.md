Claro. Abaixo está a **versão final enxuta**, mantendo o roteiro do professor **sem acrescentar seções desnecessárias**. O caso real da **Tay Podologia e Estética** aparece apenas como aplicação prática, sem envolver outros projetos ou conceitos externos.

Salve exatamente como:

`ATIVIDADE-teorica-regra-negocio-grupo-XX.md`

````markdown
# Atividade Teórica: Regra de Negócio no BD versus na Aplicação

**Aluno(s):** Charliston Alves de Amorim, Cristiano Macedo Mendes, Nilson Araújo Cosma  
**Turma:** Banco de Dados 2026  
**Data:** 18/08/2026  
**Repositório Git:** Inserir link do repositório

---

## Resumo Executivo

Regras de negócio são condições que determinam como um sistema deve funcionar para atender às necessidades de uma organização. Uma questão importante de arquitetura é definir onde essas regras devem ser implementadas: no Banco de Dados, por meio de mecanismos como `CHECK`, `UNIQUE`, `FOREIGN KEY`, triggers, stored procedures e transações, ou na aplicação, por meio de validações e regras nas camadas de serviço.

A análise realizada pelo grupo indica que não existe uma solução única para todos os casos. Regras relacionadas à integridade dos dados devem possuir proteção no Banco de Dados, enquanto regras de fluxo, comportamento e processos podem ser tratadas principalmente na aplicação.

Assim, o grupo defende uma **abordagem híbrida**, na qual Banco de Dados e aplicação possuem responsabilidades complementares.

---

# 1. Desenvolvimento Teórico

## 1.1 O que é regra de negócio?

Regra de negócio é uma condição, restrição ou comportamento que define como uma atividade deve funcionar dentro de uma organização.

Exemplos:

- CPF deve ser único;
- determinado campo é obrigatório;
- um atendimento deve estar associado a um cliente existente;
- um serviço não pode possuir valor negativo;
- determinada operação somente pode ocorrer após outra etapa ser concluída.

É importante diferenciar a **regra** de sua **implementação**.

Por exemplo:

> **Regra:** CPF deve ser único por cliente.

> **Implementação:** utilizar uma constraint `UNIQUE` no Banco de Dados.

As regras de integridade, como unicidade, obrigatoriedade e integridade referencial, possuem forte relação com o Banco de Dados. Já regras de fluxo, preço, comportamento e decisões que mudam frequentemente são normalmente mais adequadas à aplicação.

---

## 1.2 Regras no Banco de Dados

O Banco de Dados possui mecanismos próprios para proteger a consistência dos dados.

### CHECK

Permite determinar uma condição que deve ser verdadeira.

```sql
CREATE TABLE servicos (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(150) NOT NULL,
    valor NUMERIC(10,2) NOT NULL,
    CHECK (valor > 0)
);
````

Nesse caso, um serviço não pode possuir valor menor ou igual a zero.

### UNIQUE

Garante que um valor não seja duplicado.

```sql
CREATE TABLE clientes (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(150) NOT NULL,
    cpf VARCHAR(11) NOT NULL UNIQUE
);
```

Assim, o Banco de Dados impede o cadastro de dois clientes com o mesmo CPF.

### FOREIGN KEY

Garante a integridade referencial entre tabelas.

```sql
CREATE TABLE atendimentos (
    id BIGSERIAL PRIMARY KEY,
    cliente_id BIGINT NOT NULL,
    FOREIGN KEY (cliente_id)
        REFERENCES clientes(id)
);
```

Um atendimento não poderá referenciar um cliente inexistente.

### Triggers

Triggers permitem executar funções automaticamente quando determinados eventos acontecem, como `INSERT`, `UPDATE` ou `DELETE`.

Podem ser utilizadas, por exemplo, para registrar alterações ou executar determinadas ações automaticamente.

Entretanto, seu uso excessivo pode tornar o comportamento do sistema mais difícil de compreender e manter.

### Stored Procedures

Stored procedures são procedimentos armazenados no Banco de Dados e podem centralizar determinadas operações.

São úteis em situações específicas, mas seu uso excessivo pode aumentar o acoplamento da aplicação ao SGBD.

### Transações e ACID

Transações permitem tratar várias operações como uma unidade lógica.

```sql
BEGIN;

-- operação 1
-- operação 2
-- operação 3

COMMIT;
```

Em caso de falha:

```sql
ROLLBACK;
```

ACID representa:

* **Atomicidade:** a operação é realizada completamente ou desfeita;
* **Consistência:** as regras de integridade devem ser preservadas;
* **Isolamento:** operações concorrentes são controladas;
* **Durabilidade:** alterações confirmadas permanecem persistidas.

### Vantagens do Banco de Dados

* Protege os dados independentemente da aplicação utilizada.
* Centraliza regras fundamentais de integridade.
* Garante unicidade e integridade referencial.
* Possui mecanismos próprios para transações e concorrência.

### Desvantagens e limitações

* Triggers e procedures complexas podem dificultar manutenção.
* Recursos específicos podem aumentar a dependência do SGBD.
* Regras que mudam frequentemente podem ser mais difíceis de alterar.
* Fluxos complexos de negócio geralmente são mais adequados à aplicação.

---

## 1.3 Regras na Aplicação

A aplicação pode realizar validações de entrada e concentrar regras de negócio em camadas de serviço.

Exemplo:

```text
cadastrarCliente(nome, cpf):

    se nome estiver vazio:
        retornar "Nome obrigatório"

    se CPF for inválido:
        retornar "CPF inválido"

    salvar cliente

    retornar "Cliente cadastrado"
```

A aplicação é especialmente adequada para regras relacionadas a:

* validação de formulários;
* mensagens de erro;
* fluxos de negócio;
* cálculos;
* decisões;
* regras comerciais;
* comportamentos que mudam frequentemente.

### Vantagens

* Maior flexibilidade para regras complexas.
* Facilita alterações de regras que mudam frequentemente.
* Permite mensagens de erro mais amigáveis.
* Facilita testes de regras de negócio.

### Desvantagens

* Uma regra pode ser ignorada se outra aplicação acessar diretamente o Banco de Dados.
* Diferentes aplicações podem implementar a mesma regra de maneiras diferentes.
* Validações isoladas na aplicação podem apresentar problemas de concorrência.
* A consistência pode depender da correta implementação de todos os sistemas que acessam os dados.

---

## 1.4 Comparativo BD x Aplicação

| Critério                      | Banco de Dados                                               | Aplicação                              |
| ----------------------------- | ------------------------------------------------------------ | -------------------------------------- |
| **Consistência**              | Muito forte para integridade dos dados                       | Depende da implementação               |
| **Segurança**                 | Pode impedir diretamente dados inválidos                     | Valida antes da persistência           |
| **Performance**               | Eficiente para constraints e operações sobre os dados        | Flexível para regras e cálculos        |
| **Manutenção**                | Simples para constraints; pode complicar com muitas triggers | Geralmente mais flexível               |
| **Portabilidade**             | Pode depender do SGBD                                        | Pode depender da linguagem/framework   |
| **Controle central da regra** | Alto                                                         | Pode diminuir com múltiplas aplicações |
| **Integridade referencial**   | Excelente com `FOREIGN KEY`                                  | Não deve depender somente da aplicação |
| **Regras complexas**          | Possíveis, mas podem ficar difíceis de manter                | Geralmente mais adequadas              |
| **Experiência do usuário**    | Limitada                                                     | Melhor capacidade de interação         |
| **Transações**                | Possui mecanismos nativos                                    | Coordena operações utilizando o BD     |

---

## 1.5 Análise crítica: qual a melhor opção?

O grupo entende que **não existe uma opção vencedora absoluta**.

A melhor solução depende da natureza da regra.

### Sistema acessado por múltiplas aplicações

Quando diferentes sistemas acessam o mesmo Banco de Dados, regras fundamentais de integridade devem ser protegidas no próprio Banco de Dados.

Por exemplo, a regra:

> CPF deve ser único.

Não deve depender somente da validação de uma aplicação.

### Dados sensíveis ou com exigência legal/fiscal

Quanto maior o impacto de uma inconsistência, maior a necessidade de mecanismos que garantam a integridade independentemente da aplicação.

Nesse cenário, regras fundamentais devem possuir proteção no Banco de Dados, além das validações e controles existentes na aplicação.

### Regras que mudam frequentemente

Regras comerciais, promocionais ou de fluxo podem mudar com frequência. Nesse caso, a aplicação normalmente oferece maior flexibilidade para manutenção.

### Protótipos e equipes pequenas

Mesmo em projetos pequenos, constraints básicas como `PRIMARY KEY`, `NOT NULL`, `UNIQUE`, `FOREIGN KEY` e `CHECK` possuem baixo custo e podem evitar problemas futuros.

Portanto, o tamanho da equipe não elimina a importância da integridade no Banco de Dados.

---

# 2. Exemplos e Casos

## 2.1 Exemplo em PostgreSQL

Considerando um sistema de atendimento:

```sql
CREATE TABLE clientes (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(150) NOT NULL,
    cpf VARCHAR(11) NOT NULL UNIQUE
);

CREATE TABLE servicos (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(150) NOT NULL,
    valor NUMERIC(10,2) NOT NULL CHECK (valor > 0)
);

CREATE TABLE atendimentos (
    id BIGSERIAL PRIMARY KEY,
    cliente_id BIGINT NOT NULL,
    servico_id BIGINT NOT NULL,

    FOREIGN KEY (cliente_id)
        REFERENCES clientes(id),

    FOREIGN KEY (servico_id)
        REFERENCES servicos(id)
);
```

O Banco de Dados garante:

* identificação dos registros;
* campos obrigatórios;
* CPF único;
* valor positivo do serviço;
* cliente existente;
* serviço existente.

---

## 2.2 Exemplo de validação na aplicação

Uma aplicação poderia implementar:

```text
agendarAtendimento(cliente, profissional, servico, horario):

    verificar se o cliente existe

    verificar se o profissional está ativo

    verificar se o serviço está disponível

    verificar se o horário está disponível

    se houver conflito:
        retornar "Horário indisponível"

    criar agendamento

    retornar "Agendamento realizado"
```

Nesse exemplo, a aplicação controla o fluxo do processo, enquanto o Banco de Dados protege a integridade dos registros.

---

## 2.3 Caso real: Tay Podologia e Estética

A **Tay Podologia e Estética** pode ser utilizada como exemplo real de aplicação dos conceitos.

Em um sistema de atendimento da empresa, poderiam existir entidades como:

* clientes;
* profissionais;
* serviços;
* agendamentos;
* atendimentos;
* pagamentos.

Algumas regras poderiam ser distribuídas da seguinte forma:

| Regra                                      | Camada mais adequada                           |
| ------------------------------------------ | ---------------------------------------------- |
| Cliente deve possuir identificação         | Banco de Dados                                 |
| CPF deve ser único                         | Banco de Dados                                 |
| Serviço deve possuir valor válido          | Banco de Dados                                 |
| Atendimento deve possuir cliente existente | Banco de Dados                                 |
| Profissional deve estar disponível         | Aplicação                                      |
| Horário não pode possuir conflito          | Aplicação + mecanismos de BD quando necessário |
| Mensagem de erro ao usuário                | Aplicação                                      |
| Fluxo do agendamento                       | Aplicação                                      |
| Regras comerciais que mudam frequentemente | Aplicação                                      |

Esse exemplo demonstra que a escolha não precisa ser exclusivamente pelo Banco de Dados ou pela aplicação.

---

## 2.4 O dono da regra

Uma questão importante é definir **quem garante a regra**.

Se uma regra existir somente na aplicação, outro sistema poderá acessar o Banco de Dados sem passar por essa validação.

Exemplo:

```text
Aplicação → valida CPF → grava
Outro sistema → grava diretamente
```

Nesse caso, a regra pode ser violada.

Por outro lado, colocar todas as regras somente no Banco de Dados também pode gerar problemas quando regras complexas de processo forem transformadas em triggers e procedures difíceis de manter.

Outro risco é a duplicação inconsistente:

```text
Aplicação A → regra X
Aplicação B → regra X + Y
Banco → regra X + Y + Z
```

Isso pode gerar comportamentos diferentes e dados inconsistentes.

Por isso, deve existir uma definição clara de responsabilidade.

A aplicação pode **validar e orientar** o usuário, enquanto o Banco de Dados deve **garantir as invariantes fundamentais dos dados**.

---

# 3. Referências

* POSTGRESQL GLOBAL DEVELOPMENT GROUP. **PostgreSQL Documentation — Constraints**. Disponível em: [https://www.postgresql.org/docs/current/ddl-constraints.html](https://www.postgresql.org/docs/current/ddl-constraints.html)
* POSTGRESQL GLOBAL DEVELOPMENT GROUP. **PostgreSQL Documentation — CREATE TRIGGER**. Disponível em: [https://www.postgresql.org/docs/current/sql-createtrigger.html](https://www.postgresql.org/docs/current/sql-createtrigger.html)
* POSTGRESQL GLOBAL DEVELOPMENT GROUP. **PostgreSQL Documentation — User-Defined Procedures**. Disponível em: [https://www.postgresql.org/docs/current/xproc.html](https://www.postgresql.org/docs/current/xproc.html)
* POSTGRESQL GLOBAL DEVELOPMENT GROUP. **PostgreSQL Documentation — Transaction Isolation**. Disponível em: [https://www.postgresql.org/docs/current/transaction-iso.html](https://www.postgresql.org/docs/current/transaction-iso.html)
* EVANS, Eric. **Domain-Driven Design: Tackling Complexity in the Heart of Software**. Addison-Wesley, 2003.
* FOWLER, Martin. **Patterns of Enterprise Application Architecture**. Addison-Wesley, 2002.
* SOMMERVILLE, Ian. **Engenharia de Software**. Pearson.

---

# 4. Conclusões

A atividade demonstrou que regras de negócio podem ser implementadas tanto no Banco de Dados quanto na aplicação, mas cada camada possui responsabilidades e características diferentes.

O Banco de Dados é especialmente importante para garantir regras fundamentais de integridade, como:

* obrigatoriedade;
* unicidade;
* integridade referencial;
* valores válidos;
* consistência durante transações.

A aplicação é mais adequada para:

* fluxos;
* comportamentos;
* cálculos;
* decisões;
* regras complexas;
* interação com o usuário.

O grupo conclui que a melhor abordagem é **híbrida**.

A aplicação deve fornecer validações e controlar os processos de negócio, enquanto o Banco de Dados deve proteger as regras fundamentais que garantem a integridade dos dados.

Dessa forma, uma validação na aplicação melhora a experiência do usuário, mas não substitui uma constraint quando a regra precisa ser uma garantia permanente dos dados.

Portanto:

> **A aplicação orienta e executa o negócio; o Banco de Dados protege a integridade dos dados.**

A combinação adequada das duas camadas proporciona maior consistência, segurança, flexibilidade e facilidade de manutenção.

---

# Link do Repositório Git

**Repositório:** Inserir link do repositório Git do grupo

````

### Nome do arquivo

Sugiro utilizar:

```text
ATIVIDADE-teorica-regra-negocio-grupo-XX.md
````

Substituam `XX` pelo número real do grupo.

**Essa versão já está no ponto certo para a atividade:** não é superficial, responde ao roteiro inteiro e não transforma uma atividade teórica de Banco de Dados em um documento excessivamente grande.
