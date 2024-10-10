---
tags:
  - labs
  - polytech
  - practice
---
---
### ***Графика по RS-триггеру***
![[Pasted image 20241007145700.png|800]]
*Переключение из состояния хранения "1" в состояние хранения "0"*:
*~={red}R=~* = 1, ~={red}*S*=~ = 0   =>  Хранение в Q - 0 до момента переключения ~={red}*RS*=~ наоборот

![[Pasted image 20241007150751.png|720]]
*Комбинация хранения (R = 1, S = 1) - Q = 0 не изменится*

![[Pasted image 20241007151551.png|720]]
*Проверка работы при повторе прошлой комбинации переключения состояния хранения - Q **не изменилось***

![[Pasted image 20241007152049.png|700]]
*Переключение значений RS из состояния хранения "0" в состояние хранения "1"*  => Изменение сигнала Q и последующее ~={green}~={blue}хранение=~=~.
### ***Структурная модель RS-триггера***
```vhdl
LIBRARY ieee; -- Подключение библиотеки ieee
USE ieee.std_logic_1164.ALL; -- использование библиотечного модуля с доп. типами переменных
ENTITY rstr IS
PORT
(
	s : IN std_logic;
	r : IN std_logic;
	q : INOUT std_logic; -- двунаправленный сигнал (видно на картинках: одновременное использование и запись в выход)
	qb : INOUT std_logic
);
END rstr;
ARCHITECTURE behav OF rstr IS
	COMPONENT notand -- Описание используемого компонента NAND
	PORT
	(
		a : IN std_logic;
		b : IN std_logic;
		c : INOUT std_logic
	);
	END COMPONENT;
BEGIN
	u1: notand -- Указание u1, как компонента notand
	PORT MAP (s, qb, q); -- Указание для входов и выходов для u1
	u2: notand
	PORT MAP (q, r, qb);
END behav;
CONFIGURATION con OF rstr IS -- Создание новой конфигурации con
	FOR behav
		FOR u1, u2: notand
			USE ENTITY work.notand (behavior);
		END FOR;
	END FOR;
END con;
```

![[Pasted image 20241007154225.png]]
### ***Поведенческая модель работы RS-триггера***
```vhdl
LIBRARY ieee; -- Подключение библиотеки
USE ieee.std_logic_1164.ALL; -- Доп. типы переменных
ENTITY rstr1 IS -- Ввод новой сущности (интерфейса схемы)
PORT
(
	s : IN std_logic;
	r : IN std_logic;
	q : OUT std_logic
);
END rstr1;
ARCHITECTURE behav OF rstr1 IS
	SIGNAL qs: std_logic; -- создание внутреннего сигнала
BEGIN
	PROCESS (s, r) -- запуск процесса при изменении R или S
	BEGIN
		IF s='1' THEN
			IF r='1' THEN qs<=qs; -- Хранение R,S = 1,1
			ELSE qs<='0'; -- Иначе сброс триггера
			END IF;
		ELSE qs<='1'; -- Случай когда SET неактивен => qs = 1
		END IF;
	END PROCESS;
	q<=qs; -- Передача основному сигналу значения qs
END behav;
```
![[Pasted image 20241007155322.png]]
### ***Графика D-триггера (триггер - защёлка)*** 
#### Принцип работы
- Когда L = 0, триггер ~={red}игнорирует=~ состояние на входе D и сохраняет предыдущее значение на выходе Q
- Когда L = 1, триггер ~={red}копирует=~ состояние **D** на **Q** и фиксирует это значение до следующего тактового сигнала.

![[Pasted image 20241007164214.png]]
### ***Структурная модель D-триггера***
```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;
ENTITY dtr IS -- Описание интерфейса D-триггера
PORT
(
	d : IN std_logic;
	l : IN std_logic;
	q : INOUT std_logic;
	qb : INOUT std_logic
);
END dtr;
ARCHITECTURE behav OF dtr IS
	COMPONENT notand -- Описание интерфейса nand
	PORT
	(
		a : IN std_logic;
		b : IN std_logic;
		c : INOUT std_logic
	);
	END COMPONENT;
	COMPONENT rstr -- Описание интерфейса RS-триггера
	PORT
	(
		s : IN std_logic;
		r : IN std_logic;
		q : INOUT std_logic;
		qb : INOUT std_logic
	);
	END COMPONENT;

	SIGNAL u1s: std_logic; -- Промежуточный сигнал для выхода первого nand'a
	SIGNAL u2s: std_logic; -- Промежуточный сигнал для выхода второго nand'a

BEGIN
	u1: notand
	PORT MAP (d, l, u1s);
	u2: notand
	PORT MAP (u1s, l, u2s);
	rs: rstr
	PORT MAP (u1s, u2s, q, qb);
END behav;
CONFIGURATION con OF dtr IS
	FOR behav
		FOR u1, u2: notand
			USE ENTITY work.notand(behavior);
		END FOR;
		FOR rs: rstr
			USE ENTITY work.rstr(behav);
		END FOR;
	END FOR;
END con;
```
![[Pasted image 20241007165804.png|]]
### ***Поведенческая модель D-триггера***
```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;
ENTITY dtr1 IS
PORT
(
	d : IN std_logic;
	l : IN std_logic;
	q : INOUT std_logic;
	qb : INOUT std_logic
);
END dtr1;
ARCHITECTURE behav OF dtr1 IS
	SIGNAL qs: std_logic; -- внутренний сигнал для использования в PROCESS
BEGIN
	PROCESS (d, l) -- Сработает при изменении d или l
	BEGIN
		IF l='1' THEN
			qs<=d; -- Копирование состояния d на временный сигнал qs
		END IF;
	END PROCESS;
	q<=qs;
	qb<= NOT qs;
END behav;
```
![[Pasted image 20241007171132.png]]