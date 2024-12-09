# Sistema de Gerenciamento Educacional

Este sistema gerencia disciplinas, professores e turmas usando pacotes PL/SQL no Oracle.

## Criar Tabelas
 
### *ALUNO*
### *DISCIPLINA*
### *PROFESSOR*

## Pacotes

### *PKG_DISCIPLINA*
- *ADICIONAR_DISCIPLINA*: Adiciona uma nova disciplina.
- *ATUALIZAR_DISCIPLINA*: Atualiza o nome de uma disciplina existente.
- *LISTAR_DISCIPLINAS*: Lista todas as disciplinas.
- *TOTAL_TURMAS_DISCIPLINA*: Calcula o total de turmas de uma disciplina.

### *PKG_PROFESSOR*
- *ADICIONAR_PROFESSOR*: Adiciona um novo professor.
- *ATUALIZAR_PROFESSOR*: Atualiza o nome de um professor existente.
- *LISTAR_PROFESSORES*: Lista todos os professores.
- *MEDIA_TURMAS_PROFESSOR*: Calcula a média de turmas de um professor.

REM   Script: L02 - Lista de exercício Packages
REM   L0Lista de exercício Packages

CREATE TABLE Aluno ( 
    id_aluno NUMBER PRIMARY KEY, 
    nome VARCHAR2(100), 
    data_nascimento DATE, 
    id_curso NUMBER 
)
;

CREATE TABLE Disciplina ( 
    id_disciplina NUMBER PRIMARY KEY, 
    nome VARCHAR2(100), 
    descricao VARCHAR2(255), 
    carga_horaria NUMBER 
)
;

CREATE TABLE Matricula ( 
    id_matricula NUMBER PRIMARY KEY, 
    id_aluno NUMBER, 
    id_disciplina NUMBER, 
    FOREIGN KEY (id_aluno) REFERENCES Aluno(id_aluno), 
    FOREIGN KEY (id_disciplina) REFERENCES Disciplina(id_disciplina) 
)
;

CREATE TABLE Professor ( 
    id_professor NUMBER PRIMARY KEY, 
    nome VARCHAR2(100) 
)
;

CREATE TABLE Turma ( 
    id_turma NUMBER PRIMARY KEY, 
    id_professor NUMBER, 
    id_disciplina NUMBER, 
    FOREIGN KEY (id_professor) REFERENCES Professor(id_professor), 
    FOREIGN KEY (id_disciplina) REFERENCES Disciplina(id_disciplina) 
)
;

CREATE OR REPLACE PACKAGE PKG_ALUNO AS 
    PROCEDURE excluir_aluno(id_aluno NUMBER); 
    PROCEDURE listar_alunos_maiores_dezoito; 
    PROCEDURE listar_alunos_por_curso(id_curso NUMBER); 
END PKG_ALUNO; 

/

CREATE OR REPLACE PACKAGE BODY PKG_ALUNO AS 
    PROCEDURE excluir_aluno(id_aluno NUMBER) IS 
    BEGIN 
        DELETE FROM Matricula WHERE id_aluno = id_aluno; 
        DELETE FROM Aluno WHERE id_aluno = id_aluno; 
        DBMS_OUTPUT.PUT_LINE('Aluno excluído com sucesso.'); 
    END excluir_aluno; 
 
    PROCEDURE listar_alunos_maiores_dezoito IS 
        CURSOR cur_alunos IS 
            SELECT nome, data_nascimento 
            FROM Aluno 
            WHERE FLOOR(MONTHS_BETWEEN(SYSDATE, data_nascimento) / 12) > 18; 
    BEGIN 
        FOR aluno IN cur_alunos LOOP 
            DBMS_OUTPUT.PUT_LINE('Nome: ' || aluno.nome || ', Nascimento: ' || aluno.data_nascimento); 
        END LOOP; 
    END listar_alunos_maiores_dezoito; 
 
    PROCEDURE listar_alunos_por_curso(id_curso NUMBER) IS 
        CURSOR cur_alunos_curso IS 
            SELECT nome 
            FROM Aluno 
            WHERE id_curso = id_curso; 
    BEGIN 
        FOR aluno IN cur_alunos_curso LOOP 
            DBMS_OUTPUT.PUT_LINE('Nome: ' || aluno.nome); 
        END LOOP; 
    END listar_alunos_por_curso; 
END PKG_ALUNO; 

/

CREATE SEQUENCE SEQ_DISCIPLINA 
    START WITH 1 
    INCREMENT BY 1 
    NOCACHE
;

CREATE OR REPLACE PACKAGE PKG_DISCIPLINA AS 
    PROCEDURE cadastrar_disciplina(nome VARCHAR2, descricao VARCHAR2, carga_horaria NUMBER); 
    PROCEDURE listar_total_alunos_por_disciplina; 
    PROCEDURE media_idade_por_disciplina(id_disciplina NUMBER); 
    PROCEDURE listar_alunos_por_disciplina(id_disciplina NUMBER); 
END PKG_DISCIPLINA; 

/

CREATE OR REPLACE PACKAGE BODY PKG_DISCIPLINA AS 
    PROCEDURE cadastrar_disciplina(nome VARCHAR2, descricao VARCHAR2, carga_horaria NUMBER) IS 
    BEGIN 
        INSERT INTO Disciplina (id_disciplina, nome, descricao, carga_horaria) 
        VALUES (SEQ_DISCIPLINA.NEXTVAL, nome, descricao, carga_horaria); 
        DBMS_OUTPUT.PUT_LINE('Disciplina cadastrada com sucesso.'); 
    END cadastrar_disciplina; 
 
    PROCEDURE listar_total_alunos_por_disciplina IS 
        CURSOR cur_total IS 
            SELECT d.nome, COUNT(m.id_aluno) AS total_alunos 
            FROM Disciplina d 
            JOIN Matricula m ON d.id_disciplina = m.id_disciplina 
            GROUP BY d.nome 
            HAVING COUNT(m.id_aluno) > 10; 
    BEGIN 
        FOR registro IN cur_total LOOP 
            DBMS_OUTPUT.PUT_LINE('Disciplina: ' || registro.nome || ', Total de alunos: ' || registro.total_alunos); 
        END LOOP; 
    END listar_total_alunos_por_disciplina; 
 
    PROCEDURE media_idade_por_disciplina(id_disciplina NUMBER) IS 
        CURSOR cur_media IS 
            SELECT AVG(FLOOR(MONTHS_BETWEEN(SYSDATE, a.data_nascimento) / 12)) AS media_idade 
            FROM Matricula m 
            JOIN Aluno a ON m.id_aluno = a.id_aluno 
            WHERE m.id_disciplina = id_disciplina; 
    BEGIN 
        FOR registro IN cur_media LOOP 
            DBMS_OUTPUT.PUT_LINE('Média de idade: ' || registro.media_idade); 
        END LOOP; 
    END media_idade_por_disciplina; 
 
    PROCEDURE listar_alunos_por_disciplina(id_disciplina NUMBER) IS 
        CURSOR cur_alunos IS 
            SELECT a.nome 
            FROM Matricula m 
            JOIN Aluno a ON m.id_aluno = a.id_aluno 
            WHERE m.id_disciplina = id_disciplina; 
    BEGIN 
        FOR aluno IN cur_alunos LOOP 
            DBMS_OUTPUT.PUT_LINE('Nome: ' || aluno.nome); 
        END LOOP; 
    END listar_alunos_por_disciplina; 
END PKG_DISCIPLINA; 

/

CREATE OR REPLACE PACKAGE PKG_PROFESSOR AS 
    PROCEDURE listar_turmas_por_professor; 
    FUNCTION total_turmas_professor(id_professor NUMBER) RETURN NUMBER; 
    FUNCTION professor_por_disciplina(id_disciplina NUMBER) RETURN VARCHAR2; 
END PKG_PROFESSOR; 

/

CREATE OR REPLACE PACKAGE BODY PKG_PROFESSOR AS 
    PROCEDURE listar_turmas_por_professor IS 
        CURSOR cur_turmas IS 
            SELECT p.nome, COUNT(t.id_turma) AS total_turmas 
            FROM Professor p 
            JOIN Turma t ON p.id_professor = t.id_professor 
            GROUP BY p.nome 
            HAVING COUNT(t.id_turma) > 1; 
    BEGIN 
        FOR registro IN cur_turmas LOOP 
            DBMS_OUTPUT.PUT_LINE('Professor: ' || registro.nome || ', Total de turmas: ' || registro.total_turmas); 
        END LOOP; 
    END listar_turmas_por_professor; 
 
    FUNCTION total_turmas_professor(id_professor NUMBER) RETURN NUMBER IS 
        total NUMBER; 
    BEGIN 
        SELECT COUNT(*) INTO total 
        FROM Turma 
        WHERE id_professor = id_professor; 
        RETURN total; 
    END total_turmas_professor; 
 
    FUNCTION professor_por_disciplina(id_disciplina NUMBER) RETURN VARCHAR2 IS 
        nome_professor VARCHAR2(100); 
    BEGIN 
        SELECT p.nome INTO nome_professor 
        FROM Professor p 
        JOIN Turma t ON p.id_professor = t.id_professor 
        WHERE t.id_disciplina = id_disciplina; 
        RETURN nome_professor; 
    END professor_por_disciplina; 
END PKG_PROFESSOR; 

/

BEGIN 
    PKG_ALUNO.excluir_aluno(1); 
END; 

/

BEGIN 
    PKG_ALUNO.listar_alunos_maiores_dezoito; 
END; 

/

BEGIN 
    PKG_ALUNO.listar_alunos_por_curso(1); 
END; 

/

BEGIN 
    PKG_DISCIPLINA.listar_total_alunos_por_disciplina; 
END; 

/

BEGIN 
    PKG_DISCIPLINA.media_idade_por_disciplina(1); 
END; 

/

BEGIN 
    PKG_DISCIPLINA.listar_alunos_por_disciplina(1); 
END; 

/

BEGIN 
    PKG_PROFESSOR.listar_turmas_por_professor; 
END; 

/

BEGIN 
    DBMS_OUTPUT.PUT_LINE('Total de turmas: ' || PKG_PROFESSOR.total_turmas_professor(1)); 
END; 

/

BEGIN 
    DBMS_OUTPUT.PUT_LINE('Professor: ' || PKG_PROFESSOR.professor_por_disciplina(1)); 
END; 

/

BEGIN 
    PKG_PROFESSOR.listar_turmas_por_professor; 
END; 

/

BEGIN 
    DBMS_OUTPUT.PUT_LINE('Total de turmas: ' || PKG_PROFESSOR.total_turmas_professor(1)); 
END; 

/

