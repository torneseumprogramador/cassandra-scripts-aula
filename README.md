# install cassandra MacOs
https://medium.com/@manishyadavv/how-to-install-cassandra-on-mac-os-d9338fcfcba4

# install
brew install cassandra

# start cassandra
cassandra -f

# acessar no terminal CLI
cqlsh

# acessar com usuário e senha
cqlsh 127.0.0.1 -u cassandra -p cassandra

# documentação cassandra
https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useAboutCQL.html

# create database(keyspace) cassandra
https://www.geeksforgeeks.org/create-database-in-cassandra/

### partition key = primary key
- partição criada organizando itens clusterizados 
- para criar mais de uma partição em uma tabela utilizar o esquema "primary key((cpf,telefone), nome)" neste caso, cpf e telefone é partition key combinados e nome é cluester key
### clustering key = itens na criação da tabela para Ordenação 


# new cassandra create keyspace
https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useExampleCreatingKeyspace.html

# scrit create 
CREATE KEYSPACE IF NOT EXISTS App_data_danilo WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 3 };

# verificar databases criadas
describe keyspaces; 

# describe uma database
describe App_data_danilo; 

# usar o database
USE App_data_danilo;

# create table
https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useCreateTable.html
CREATE TABLE App_data_danilo.clientes ( id UUID PRIMARY KEY, nome text, data_nasc timestamp, nacionalidade text, peso text, altura text );

# insert data
INSERT INTO App_data_danilo.clientes (id, nome, nacionalidade) VALUES (5b6962dd-3f90-4c93-8f61-eabfa4a803e2, 'Danilo','Brasileiro');
INSERT INTO App_data_danilo.calendar (race_id, race_start_date, race_end_date, race_name) VALUES 
  (201, '2015-02-18', '2015-02-22', $$Women's Tour of New Zealand$$);

# exercicio criar atualizar apagar
# criar tabela
cqlsh> CREATE TABLE App_data_danilo.cyclist_expenses ( 
  cyclist_name text, 
  balance float STATIC, 
  expense_id int, 
  amount float, 
  description text, 
  paid boolean, 
  PRIMARY KEY (cyclist_name, expense_id) 
);

# insert
cqlsh> BEGIN BATCH
  INSERT INTO App_data_danilo.cyclist_expenses (cyclist_name, balance) VALUES ('Vera ADRIAN', 0) IF NOT EXISTS;
  INSERT INTO App_data_danilo.cyclist_expenses (cyclist_name, expense_id, amount, description, paid) VALUES ('Vera ADRIAN', 1, 7.95, 'Breakfast', false);
  APPLY BATCH;

# update
cqlsh> UPDATE App_data_danilo.cyclist_expenses SET balance = -7.95 WHERE cyclist_name = 'Vera ADRIAN' IF balance = 0;

# update com transaction
cqlsh> BEGIN BATCH
INSERT INTO App_data_danilo.cyclist_expenses (cyclist_name, expense_id, amount, description, paid) VALUES ('Vera ADRIAN', 2, 13.44, 'Lunch', true);
INSERT INTO App_data_danilo.cyclist_expenses (cyclist_name, expense_id, amount, description, paid) VALUES ('Vera ADRIAN', 3, 25.00, 'Dinner', false);
UPDATE App_data_danilo.cyclist_expenses SET balance = -32.95 WHERE cyclist_name = 'Vera ADRIAN' IF balance = -7.95;
APPLY BATCH;

# + update
cqlsh> BEGIN BATCH
UPDATE App_data_danilo.cyclist_expenses SET balance = 0 WHERE cyclist_name = 'Vera ADRIAN' IF balance = -32.95;
UPDATE App_data_danilo.cyclist_expenses SET paid = true WHERE cyclist_name = 'Vera ADRIAN' AND expense_id = 1 IF paid = false;
UPDATE App_data_danilo.cyclist_expenses SET paid = true WHERE cyclist_name = 'Vera ADRIAN' AND expense_id = 3 IF paid = false;
APPLY BATCH;

# + insert
cqlsh> BEGIN LOGGED BATCH
INSERT INTO App_data_danilo.cyclist_names (cyclist_name, race_id) VALUES ('Vera ADRIAN', 100);
INSERT INTO App_data_danilo.cyclist_by_id (race_id, cyclist_name) VALUES (100, 'Vera ADRIAN');
APPLY BATCH;


# exercicio de select
# tabela
CREATE TABLE App_data_danilo.rank_by_year_and_name ( 
  race_year int, 
  race_name text, 
  cyclist_name text, 
  rank int, 
  PRIMARY KEY ((race_year, race_name), rank) );

# insert
INSERT INTO App_data_danilo.rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Tour of Japan - Stage 4 - Minami > Shinshu', 'Benjamin PRADES', 1);
INSERT INTO App_data_danilo.rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Tour of Japan - Stage 4 - Minami > Shinshu', 'Adam PHELAN', 2);
INSERT INTO App_data_danilo.rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Tour of Japan - Stage 4 - Minami > Shinshu', 'Thomas LEBAS', 3);
INSERT INTO App_data_danilo.rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Giro d''Italia - Stage 11 - Forli > Imola', 'Ilnur ZAKARIN', 1);
INSERT INTO App_data_danilo.rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Giro d''Italia - Stage 11 - Forli > Imola', 'Carlos BETANCUR', 2);
INSERT INTO App_data_danilo.rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2014, '4th Tour of Beijing', 'Phillippe GILBERT', 1);
INSERT INTO App_data_danilo.rank_by_year_and_name (race_year, race_name, cyclist_name, rank)  
   VALUES (2014, '4th Tour of Beijing', 'Daniel MARTIN', 2);
INSERT INTO App_data_danilo.rank_by_year_and_name (race_year, race_name, cyclist_name, rank)  
   VALUES (2014, '4th Tour of Beijing', 'Johan Esteban CHAVES', 3);

# select 
cqlsh> SELECT * FROM App_data_danilo.rank_by_year_and_name;

# select com filtro
cqlsh> SELECT * FROM App_data_danilo.rank_by_year_and_name WHERE cyclist_name = 'Phillippe GILBERT';
# create select com filtro
cqlsh> CREATE TABLE App_data_danilo.cyclist_cat_pts ( category text, points int, id UUID,lastname text, PRIMARY KEY (category, points) ); 
INSERT INTO App_data_danilo.cyclist_cat_pts (category, points, id, lastname) 
   VALUES ('Primaria', 1, 5b6962dd-3f90-4c93-8f61-eabfa4a803e2, 'Santos');
SELECT * FROM App_data_danilo.cyclist_cat_pts WHERE category = 'GC' ORDER BY points ASC;


# select com campo especídico
cqlsh> SELECT race_name FROM App_data_danilo.rank_by_year_and_name;

# select com limit
cqlsh> SELECT race_name FROM App_data_danilo.rank_by_year_and_name limit 3;

# create select com filtro
cqlsh> CREATE TABLE App_data_danilo.cyclist_cat_pts ( category text, points int, id UUID,lastname text, PRIMARY KEY (category, points) ); 
INSERT INTO App_data_danilo.cyclist_cat_pts (category, points, id, lastname) 
   VALUES ('Primaria', 1, 5b6962dd-3f90-4c93-8f61-eabfa4a803e2, 'Santos');
SELECT * FROM App_data_danilo.cyclist_cat_pts WHERE category = 'GC' ORDER BY points ASC;

# select renomeando coluna
cqlsh> SELECT race_name, point_id, lat_long AS CITY_LATITUDE_LONGITUDE FROM cycling.route;

# alterando coluna da tabela
cqlsh> ALTER TABLE cycling.cyclist_alt_stats ADD age int;

cqlsh> ALTER TABLE cycling.cyclist_alt_stats ADD favorite_color varchar;
ALTER TABLE cycling.cyclist_alt_stats ALTER favorite_color TYPE text;

# criando indice

cqlsh> CREATE TABLE cycling.rank_by_year_and_name ( 
  race_year int, 
  race_name text, 
  cyclist_name text, 
  rank int, 
  PRIMARY KEY ((race_year, race_name), rank) 
);

cqlsh> SELECT * FROM cycling.rank_by_year_and_name WHERE race_year=2015 AND race_name='Tour of Japan - Stage 4 - Minami > Shinshu';


cqlsh> SELECT * FROM cycling.rank_by_year_and_name WHERE race_year=2015;

# criando index
cqlsh> CREATE INDEX ryear ON cycling.rank_by_year_and_name (race_year);
SELECT * FROM cycling.rank_by_year_and_name WHERE race_year=2015;

