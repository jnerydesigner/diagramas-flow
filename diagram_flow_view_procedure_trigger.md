1. **Procedure** (P) insere dados na tabela Funcionarios (F).
2. O **Trigger** (T) é disparado após cada INSERT em Funcionarios e insere dados em Auditoria (A).
3. A **View** (V) consulta os dados de Funcionarios para mostrar a média salarial por cidade.

![Texto Alternativo](docs/diagrams/diagram_flow_view_procedure_trigger.png)

## Criação da Tabela Funcionarios


```
CREATE TABLE Funcionarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100),
    cargo VARCHAR(50),
    salario DECIMAL(10,2),
    cidade VARCHAR(50),
    data_admissao DATE
);
```

## Criação da Tabela Auditoria

```
CREATE TABLE Auditoria (
    id INT AUTO_INCREMENT PRIMARY KEY,
    acao VARCHAR(255),
    data_execucao DATETIME,
    usuario VARCHAR(50)
);
```

## Trigger para Auditoria

```
CREATE TRIGGER trg_insercao_funcionarios
AFTER INSERT ON Funcionarios
FOR EACH ROW
BEGIN
    INSERT INTO Auditoria (acao, data_execucao, usuario)
    VALUES (CONCAT('Novo funcionário inserido: ', NEW.nome),
            NOW(),
            CURRENT_USER());
END;
```

### O que acontece aqui?
1. AFTER INSERT → executa depois de inserir um funcionário.

2. NEW.nome → acessa o valor do campo nome da nova tupla inserida.

3. CURRENT_USER() → registra o usuário do banco que fez a inserção.

## Procedure para Inserir Funcionários
```
CREATE PROCEDURE InserirFuncionario(
    IN p_nome VARCHAR(100),
    IN p_cargo VARCHAR(50),
    IN p_salario DECIMAL(10,2),
    IN p_cidade VARCHAR(50),
    IN p_data_admissao DATE
)
BEGIN
    INSERT INTO Funcionarios (nome, cargo, salario, cidade, data_admissao)
    VALUES (p_nome, p_cargo, p_salario, p_cidade, p_data_admissao);
END;
```

### Como usar:

```
CALL InserirFuncionario('Ana Paula', 'Analista', 4500.00, 'São Paulo', '2025-07-22');

```
## View de Estatísticas de Salário por Cidade

```
CREATE VIEW SalarioMedioPorCidade AS
SELECT cidade, AVG(salario) AS media_salarial, COUNT(*) AS total_funcionarios
FROM Funcionarios
GROUP BY cidade;

```

### Como usar
```
SELECT * FROM SalarioMedioPorCidade;
```


# Fluxo Completo
1. Executamos a procedure para adicionar um novo funcionário:
```
CALL InserirFuncionario('Carla Mendes', 'Gerente', 9000.00, 'Curitiba', '2025-07-22');

```
2. O trigger é disparado automaticamente, registrando na tabela Auditoria algo como: "Novo funcionário inserido: Carla Mendes".

3. Podemos consultar estatísticas com a view:
```
SELECT * FROM SalarioMedioPorCidade;


```