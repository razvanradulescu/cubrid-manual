****************************
Aggregate/Analysis Functions
****************************

**Aggregate Function**

Aggregate function returns one result based on the group of rows. When the **GROUP BY** clause is included, a one-row aggregate result per group is returned. When the **GROUP BY** clause is omitted, a one-row aggregate result for all rows is returned. The **HAVING** clause is used to add a condition to the query which contains the **GROUP BY** clause.

Most aggregate functions can use **DISTINCT**. For the **GROUP BY ... HAVING** clause, see `GROUP BY ... HAVING clause <#syntax_syntax_retreive_group_htm>`_.

**Analytic Function**

Analytic function calculates the aggregate value based on the result of rows. The analytic function is different from the aggregate function since it can return one or more rows based on the groups specified by the *query_partition_clause* after the **OVER** clause (when this clause is omitted, all rows are regarded as a group).

The analytic function is used along with a new analytic clause, **OVER**, for the existing aggregate functions to allow a variety of statistics for a group of specific rows. ::

	function_name ( [argument_list ] ) OVER (<analytic_clause>)
	 
	<analytic_clause>::=
		 [ <query_partition_clause> ] [ <order_by_clause> ]
		
	<query_partition_clause>::=
		PARTITION BY value_expr [, value_expr ]...
	 
	<order_by_clause>::=
		ORDER BY { expr | position | column_alias } [ ASC | DESC ]
			[, { expr | position | column_alias } [ ASC | DESC ] ] ...

*   <*query_partition_clause*> : Groups based on one or more *value_expr*. It uses the **PARTITION BY** clause to partition the query result.

*   <*order_by_clause*> : defines the data sorting method in the partition made by <*query_partition_clause*>. The result can be sorted with several keys. <When *query_partition_clause*> is omitted, the data is sorted within the overall result sets. Based on the sorting order, the function is applied to the column values of accumulated records, including the previous values.

.. function:: AVG ( [ { DISTINCT | DISTINCTROW } | UNIQUE | ALL ] expression )

	The **AVG** function calculates the arithmetic average of the value of an expression representing all rows. Only one *expression* is specified as a parameter. You can get the average without duplicates by using the **DISTINCT** or **UNIQUE** keyword in front of the expression or the average of all values by omitting the keyword or by using **ALL**.

	:param expression: Specifies an expression that returns a numeric value. An expression that returns a collection-type data is not allowed.
	:param ALL: Calculates an average value for all data (default).
	:param DISTINCT, UNIQUE: Calculates an average value without duplicates.
	:rtype: DOUBLE

	The following example shows how to retrieve the average number of gold medals that Korea won in Olympics in the *demodb* database.

	.. code-block:: sql
	
		SELECT AVG(gold)
		FROM participant
		WHERE nation_code = 'KOR'; 
						 avg(gold)
		==========================
			 9.600000000000000e+00

	The following example shows how to output the number of gold medals by year and the average number of accumulated gold medals in history, acquired whose nation_code starts with 'AU'.

	.. code-block:: sql

		SELECT host_year, nation_code, gold,
		AVG(gold) OVER (PARTITION BY nation_code ORDER BY host_year) avg_gold
		FROM participant WHERE nation_code like 'AU%';
		 
			host_year  nation_code                  gold               avg_gold
		=======================================================================
				 1988  'AUS'                           3  3.000000000000000e+00
				 1992  'AUS'                           7  5.000000000000000e+00
				 1996  'AUS'                           9  6.333333333333333e+00
				 2000  'AUS'                          16  8.750000000000000e+00
				 2004  'AUS'                          17  1.040000000000000e+01
				 1988  'AUT'                           1  1.000000000000000e+00
				 1992  'AUT'                           0  5.000000000000000e-01
				 1996  'AUT'                           0  3.333333333333333e-01
				 2000  'AUT'                           2  7.500000000000000e-01
				 2004  'AUT'                           2  1.000000000000000e+00

	The following example is removing the "ORDER BY host_year" clause under the **OVER** analysis clause from the above example. The avg_gold value is the average of gold medals for all years, so the value is identical for every year by nation_code.

	.. code-block:: sql
	
		SELECT host_year, nation_code, gold, AVG(gold) OVER (PARTITION BY nation_code) avg_gold
		FROM participant WHERE nation_code LIKE 'AU%';
		 
			host_year  nation_code                  gold                  avg_gold
		==========================================================================
				 2004  'AUS'                          17     1.040000000000000e+01
				 2000  'AUS'                          16     1.040000000000000e+01
				 1996  'AUS'                           9     1.040000000000000e+01
				 1992  'AUS'                           7     1.040000000000000e+01
				 1988  'AUS'                           3     1.040000000000000e+01
				 2004  'AUT'                           2     1.000000000000000e+00
				 2000  'AUT'                           2     1.000000000000000e+00
				 1996  'AUT'                           0     1.000000000000000e+00
				 1992  'AUT'                           0     1.000000000000000e+00
				 1988  'AUT'                           1     1.000000000000000e+00

.. function:: COUNT ( * | [ { DISTINCT | DISTINCTROW } | UNIQUE | ALL ] expression )

	The **COUNT** function returns the number of of rows returned by a query. If an asterisk (*) is specified, the number of all rows satisfying the condition (including the rows with the **NULL** value) is returned. If the **DISTINCT** or **UNIQUE** keyword is specified in front of the expression, only the number of rows that have a unique value (excluding the rows with the **NULL** value) is returned after duplicates have been removed. Therefore, the value returned is always an integer and **NULL** is never returned.

	:param expression: Specifies an expression.
	:param ALL: Gets the number of rows given in the *expression* (default).
	:param DISTINCT, UNIQUE: Gets the number of rows without duplicates.
	:rtype: INT
	
	A column that has collection type and object domain (user-defined class or multimedia class) can also be specified in the *expression*.

	The following example shows how to retrieve the number of Olympic Games that have a mascot in the *demodb* database.

	.. code-block:: sql

		SELECT COUNT(*)
		FROM olympic
		WHERE mascot IS NOT NULL; 
			 count(*)
		=============
					9

	The following example shows how to output the number of players whose nation_code is 'AUT' in *demodb* by accumulating the number of events when the event is changed. The last row shows the number of all players.

	.. code-block:: sql
	
		SELECT nation_code, event,name, COUNT(*) OVER (ORDER BY event) co
		FROM athlete WHERE nation_code='AUT';
		   nation_code           event                 name                           co
		===============================================================================
		  'AUT'                 'Athletics'           'Kiesl Theresia'                2
		  'AUT'                 'Athletics'           'Graf Stephanie'                2
		  'AUT'                 'Equestrian'          'Boor Boris'                    6
		  'AUT'                 'Equestrian'          'Fruhmann Thomas'               6
		  'AUT'                 'Equestrian'          'Munzner Joerg'                 6
		  'AUT'                 'Equestrian'          'Simon Hugo'                    6
		  'AUT'                 'Judo'                'Heill Claudia'                 9
		  'AUT'                 'Judo'                'Seisenbacher Peter'            9
		  'AUT'                 'Judo'                'Hartl Roswitha'                9
		  'AUT'                 'Rowing'              'Jonke Arnold'                 11
		  'AUT'                 'Rowing'              'Zerbst Christoph'             11
		  'AUT'                 'Sailing'             'Hagara Roman'                 15
		  'AUT'                 'Sailing'             'Steinacher Hans Peter'        15
		  'AUT'                 'Sailing'             'Sieber Christoph'             15
		  'AUT'                 'Sailing'             'Geritzer Andreas'             15
		  'AUT'                 'Shooting'            'Waibel Wolfram Jr.'           17
		  'AUT'                 'Shooting'            'Planer Christian'             17
		  'AUT'                 'Swimming'            'Rogan Markus'                 18

.. function:: DENSE_RANK() OVER ( [partition_by_clause] [order_by_clause] )

	The rank of the value in the column value group made by the **PARTITION BY** clause is calculated and output as **INTEGER**. It is used as an analytic function only. Even when there is the same rank, 1 is added to the next rank value. For example, when there are three rows of Rank 13, the next rank is 14, not 16. On the contrary, the :func:`RANK` function calculates the next rank by adding the number of same ranks.

	:rtype: INT
	
	The following example shows output of the number of Olympic gold medals of each country and the rank of the countries by year: The number of the same rank is ignored and the next rank is calculated by adding 1 to the rank.

	.. code-block:: sql
	
		SELECT host_year, nation_code, gold,
		DENSE_RANK() OVER (PARTITION BY host_year ORDER BY gold DESC) AS d_rank
		FROM participant;
		 
		host_year  nation_code                  gold       d_rank
		=============================================================
				 1988  'URS'                          55            1
				 1988  'GDR'                          37            2
				 1988  'USA'                          36            3
				 1988  'KOR'                          12            4
				 1988  'HUN'                          11            5
				 1988  'FRG'                          11            5
				 1988  'BUL'                          10            6
				 1988  'ROU'                           7            7
				 1988  'ITA'                           6            8
				 1988  'FRA'                           6            8
				 1988  'KEN'                           5            9
				 1988  'GBR'                           5            9
				 1988  'CHN'                           5            9
		...
				 1988  'CHI'                           0           14
				 1988  'ARG'                           0           14
				 1988  'JAM'                           0           14
				 1988  'SUI'                           0           14
				 1988  'SWE'                           0           14
				 1992  'EUN'                          45            1
				 1992  'USA'                          37            2
				 1992  'GER'                          33            3
		...
				 2000  'RSA'                           0           15
				 2000  'NGR'                           0           15
				 2000  'JAM'                           0           15
				 2000  'BRA'                           0           15
				 2004  'USA'                          36            1
				 2004  'CHN'                          32            2
				 2004  'RUS'                          27            3
				 2004  'AUS'                          17            4
				 2004  'JPN'                          16            5
				 2004  'GER'                          13            6
				 2004  'FRA'                          11            7
				 2004  'ITA'                          10            8
				 2004  'UKR'                           9            9
				 2004  'CUB'                           9            9
				 2004  'GBR'                           9            9
				 2004  'KOR'                           9            9
		...
				 2004  'EST'                           0           17
				 2004  'SLO'                           0           17
				 2004  'SCG'                           0           17
				 2004  'FIN'                           0           17
				 2004  'POR'                           0           17
				 2004  'MEX'                           0           17
				 2004  'LAT'                           0           17
				 2004  'PRK'                           0           17

.. function:: GROUP_CONCAT([DISTINCT] {col | expression} [ORDER BY {col | unsigned_int} [ASC | DESC]] [SEPARATOR str_val])

	The **GROUP_CONCAT** function connects the values that are not **NULL** in the group and returns the character string in the **VARCHAR** type. If there are no rows of query result or there are only **NULL** values, **NULL** will be returned.

	:param expression: Operation returning numerical values or character strings
	:param str_val: Character string to use as a separator
	:param DISTINCT: Removes duplicate values from the result.
	:param ORDER BY: Specifies the order of result values.
	:param SEPARATOR: Specifies the separator to divide the result values. If it is omitted, the default character, comma (,) will be used as a separator.
	:rtype: STRING

	The maximum size of the return value follows the configuration of the system parameter, **group_concat_max_len**. The default is **1024** bytes, the minimum value is 4 bytes and the maximum value is 33,554,432 bytes. If it exceeds the maximum value, **NULL** will be returned.

	To remove the duplicate values, use the **DISTINCT** clause. The default separator for the group result values is comma (,). To represent the separator explicitly, add the character string to use as a separator in the **SEPARATOR** clause and after that. If you want to remove separators, enter empty strings after the **SEPARATOR** clause.

	If the non-character string type is passed to the result character string, an error will be returned.

	To use the **GROUP_CONCAT** function, you must meet the following conditions.

	*   Only one expression (or a column) is allowed for an input parameter.
	*   Sorting with **ORDER BY** is available only in the the expression used as a parameter.
	*   The character string used as a separator allows not only character string type but also allows other types.

	.. code-block:: sql

		SELECT GROUP_CONCAT(s_name) FROM code;
		  group_concat(s_name)
		======================
		  'X,W,M,B,S,G'
		 
		SELECT GROUP_CONCAT(s_name ORDER BY s_name SEPARATOR ':') from code;
		  group_concat(s_name order by s_name separator ':')
		======================
		  'B:G:M:S:W:X'
		 
		CREATE TABLE t(i int);
		INSERT INTO t VALUES (4),(2),(3),(6),(1),(5);
		 
		SELECT GROUP_CONCAT(i*2+1 ORDER BY 1 SEPARATOR '') FROM t;
		  group_concat(i*2+1 order by 1 separator '')
		======================
		  '35791113'

.. function:: MAX ( [ { DISTINCT | DISTINCTROW } | UNIQUE | ALL ] expression )

	The **MAX** function gets the greatest value of expressions of all rows. Only one *expression* is specified.

	:param expression: Specifies an expression that returns a numeric or string value. An expression that returns a collection-type data is not allowed.
	:param ALL: Gets the maximum value for all data (default).
	:param DISTINCT, UNIQUE: Gets the maximum value without duplicates.
	:rtype: same type as that the expression

	For expressions that return character strings, the string that appears later in alphabetical order becomes the maximum value; for those that return numbers, the greatest value becomes the maximum value.

	The following example shows how to retrieve the maximum number of gold (*gold*) medals that Korea won in the Olympics in the *demodb* database.

	.. code-block:: sql
	
		SELECT MAX(gold) FROM participant WHERE nation_code = 'KOR';
			max(gold)
		=============
				   12

	The following example shows how to output the number of gold medals by year and the maximum number of gold medals in history, acquired by the country whose nation_code code starts with 'AU'.

	.. code-block:: sql
	
		SELECT host_year, nation_code, gold,
		MAX(gold) OVER (PARTITION BY nation_code) mx_gold
		FROM participant WHERE nation_code like 'AU%' ORDER BY nation_code, host_year;
		 
			host_year  nation_code                  gold      mx_gold
		=============================================================
				 1988  'AUS'                           3           17
				 1992  'AUS'                           7           17
				 1996  'AUS'                           9           17
				 2000  'AUS'                          16           17
				 2004  'AUS'                          17           17
				 1988  'AUT'                           1            2
				 1992  'AUT'                           0            2
				 1996  'AUT'                           0            2
				 2000  'AUT'                           2            2
				 2004  'AUT'                           2            2

.. function:: MIN ( [ { DISTINCT | DISTINCTROW } | UNIQUE | ALL ] expression )

	The **MIN** function gets the smallest value of expressions of all rows. Only one *expression* is specified. For expressions that return character strings, the string that appears earlier in alphabetical order becomes the minimum value; for those that return numbers, the smallest value becomes the minimum value.

	:param expression: Specifies an expression that returns a numeric or string value. A collection expression cannot be specified.
	:param ALL: Gets the minimum value for all data (default).
	:param DISTINCT, UNIQUE: Gets the maximum value without duplicates.
	:rtype: same type as that the expression

	The following example shows how to retrive the minimum number of gold (*gold*) medals that Korea won in the Olympics in the *demodb* database.

	.. code-block:: sql
	
		SELECT MIN(gold) FROM participant WHERE nation_code = 'KOR';
			min(gold)
		=============
					7

	The following example shows how to output the number of gold medals by year and the maximum number of gold medals in history, acquired by the country whose nation_code code starts with 'AU'.

	.. code-block:: sql

		SELECT host_year, nation_code, gold,
		MIN(gold) OVER (PARTITION BY nation_code) mn_gold
		FROM participant WHERE nation_code like 'AU%' ORDER BY nation_code, host_year;
		 
			host_year  nation_code                  gold      mn_gold
		=============================================================
				 1988  'AUS'                           3            3
				 1992  'AUS'                           7            3
				 1996  'AUS'                           9            3
				 2000  'AUS'                          16            3
				 2004  'AUS'                          17            3
				 1988  'AUT'                           1            0
				 1992  'AUT'                           0            0
				 1996  'AUT'                           0            0
				 2000  'AUT'                           2            0
				 2004  'AUT'                           2            0


.. function:: RANK() OVER ( [partition_by_clause] [order_by_clause] )

	The rank of the value in the column value group made by the **PARTITION BY** clause is calculated and output as **INTEGER**. It is used as an analytic function only. When there is another identical rank, the next rank is the number adding the number of the same ranks. For example, when there are three rows of Rank 13, the next rank is 16, not 14. On the contrary, the :func:`DENSE_RANK` function calculates the next rank by adding 1 to the rank.

	:rtype: INT

	The following example shows output of the number of Olympic gold medals of each country and the rank of the countries by year. The next rank of the same rank is calculated by adding the number of the same ranks.

	.. code-block:: sql
	
		SELECT host_year, nation_code, gold,
		RANK() OVER (PARTITION BY host_year ORDER BY gold DESC) AS g_rank
		FROM participant;
		 
			host_year  nation_code                  gold       g_rank
		=============================================================
				 1988  'URS'                          55            1
				 1988  'GDR'                          37            2
				 1988  'USA'                          36            3
				 1988  'KOR'                          12            4
				 1988  'HUN'                          11            5
				 1988  'FRG'                          11            5
				 1988  'BUL'                          10            7
				 1988  'ROU'                           7            8
				 1988  'ITA'                           6            9
				 1988  'FRA'                           6            9
				 1988  'KEN'                           5           11
				 1988  'GBR'                           5           11
				 1988  'CHN'                           5           11
		...
				 1988  'CHI'                           0           32
				 1988  'ARG'                           0           32
				 1988  'JAM'                           0           32
				 1988  'SUI'                           0           32
				 1988  'SWE'                           0           32
				 1992  'EUN'                          45            1
				 1992  'USA'                          37            2
				 1992  'GER'                          33            3
		...
				 2000  'RSA'                           0           52
				 2000  'NGR'                           0           52
				 2000  'JAM'                           0           52
				 2000  'BRA'                           0           52
				 2004  'USA'                          36            1
				 2004  'CHN'                          32            2
				 2004  'RUS'                          27            3
				 2004  'AUS'                          17            4
				 2004  'JPN'                          16            5
				 2004  'GER'                          13            6
				 2004  'FRA'                          11            7
				 2004  'ITA'                          10            8
				 2004  'UKR'                           9            9
				 2004  'CUB'                           9            9
				 2004  'GBR'                           9            9
				 2004  'KOR'                           9            9
		...
				 2004  'EST'                           0           57
				 2004  'SLO'                           0           57
				 2004  'SCG'                           0           57
				 2004  'FIN'                           0           57
				 2004  'POR'                           0           57
				 2004  'MEX'                           0           57
				 2004  'LAT'                           0           57
				 2004  'PRK'                           0           57

.. function:: ROW_NUMBER() OVER ( [partition_by_clause] [order_by_clause] )

	The rank of a row is one plus the number of distinct ranks that come before the row in question by using the **PARTITION BY** clause and outputs as **INTEGER** and it is used as the analytic function only.

	:rtype: INT

	The following example shows output of the serial number according to the number of Olympic gold medals of each country by year. If the number of gold medals is the same, the sorting follows the alphabetic order of the nation_code.

	.. code-block:: sql
	
		SELECT host_year, nation_code, gold,
		ROW_NUMBER() OVER (PARTITION BY host_year ORDER BY gold DESC) AS r_num
		FROM participant;
		 
			host_year  nation_code                  gold       r_num
		=============================================================
				 1988  'URS'                          55            1
				 1988  'GDR'                          37            2
				 1988  'USA'                          36            3
				 1988  'KOR'                          12            4
				 1988  'FRG'                          11            5
				 1988  'HUN'                          11            6
				 1988  'BUL'                          10            7
				 1988  'ROU'                           7            8
				 1988  'FRA'                           6            9
				 1988  'ITA'                           6           10
				 1988  'CHN'                           5           11
		...
				 1988  'YEM'                           0          152
				 1988  'YMD'                           0          153
				 1988  'ZAI'                           0          154
				 1988  'ZAM'                           0          155
				 1988  'ZIM'                           0          156
				 1992  'EUN'                          45            1
				 1992  'USA'                          37            2
				 1992  'GER'                          33            3
		...
				 2000  'VIN'                           0          194
				 2000  'YEM'                           0          195
				 2000  'ZAM'                           0          196
				 2000  'ZIM'                           0          197
				 2004  'USA'                          36            1
				 2004  'CHN'                          32            2
				 2004  'RUS'                          27            3
				 2004  'AUS'                          17            4
				 2004  'JPN'                          16            5
				 2004  'GER'                          13            6
				 2004  'FRA'                          11            7
				 2004  'ITA'                          10            8
				 2004  'CUB'                           9            9
				 2004  'GBR'                           9           10
				 2004  'KOR'                           9           11
		...
				 2004  'UGA'                           0          195
				 2004  'URU'                           0          196
				 2004  'VAN'                           0          197
				 2004  'VEN'                           0          198
				 2004  'VIE'                           0          199
				 2004  'VIN'                           0          200
				 2004  'YEM'                           0          201
				 2004  'ZAM'                           0          202

.. function:: STDDEV( [ { DISTINCT | DISTINCTROW } | UNIQUE | ALL] expression )
.. function:: STDDEV_POP( [ { DISTINCT | DISTINCTROW } | UNIQUE | ALL] expression )

	The functions **STDDEV** and **STDDEV_POP** are used interchangeably and they return a standard variance of the values calculated for all rows. The **STDDEV_POP** function is a standard of the SQL:1999. Only one *expression* is specified as a parameter. If the **DISTINCT** or **UNIQUE** keyword is inserted before the expression, they calculate the sample standard variance after deleting duplicates; if keyword is omitted or **ALL**, they it calculate the sample standard variance for all values.

	:param expression: Specifies an expression that returns a numeric value.
	:param ALL: Calculates the standard variance for all data (default).
	:param DISTINCT, UNIQUE: Calculates the standard variance without duplicates.
	:rtype: DOUBLE

	The return value is the same with the square root of it's variance (the return value of :func:`VAR_POP` and it is a **DOUBLE** type. If there are no rows that can be used for calculating a result, **NULL** is returned.

	The following is a formula that is applied to the function.

	.. math:: STDDEV_POP = [ (1/N) * SUM( { xI - AVG(x) }^2) ]^1/2

	.. note:: In CUBRID 2008 R3.1 or earlier, the **STDDEV** function worked the same as the :func:`STDDEV_SAMP`.

	The following example shows how to output the population standard variance of all students for all subjects.

	.. code-block:: sql
		
		CREATE TABLE student (name VARCHAR(32), subjects_id INT, score DOUBLE);
		INSERT INTO student VALUES
		('Jane',1, 78),
		('Jane',2, 50),
		('Jane',3, 60),
		('Bruce', 1, 63),
		('Bruce', 2, 50),
		('Bruce', 3, 80),
		('Lee', 1, 85),
		('Lee', 2, 88),
		('Lee', 3, 93),
		('Wane', 1, 32),
		('Wane', 2, 42),
		('Wane', 3, 99),
		('Sara', 1, 17),
		('Sara', 2, 55),
		('Sara', 3, 43);
		 
		SELECT STDDEV_POP(score) FROM student;
		 
				 stddev_pop(score)
		==========================
			 2.329711474744362e+01

	The following example shows how to output the score and population standard variance of all students by subject (subjects_id).

	.. code-block:: sql	

		SELECT subjects_id, name, score, STDDEV_POP(score) OVER(PARTITION BY subjects_id) std_pop FROM student ORDER BY subjects_id, name;
		 
		  subjects_id  name                                     score                   std_pop
		=======================================================================================
					1  'Bruce'                  6.300000000000000e+01     2.632869157402243e+01
					1  'Jane'                   7.800000000000000e+01     2.632869157402243e+01
					1  'Lee'                    8.500000000000000e+01     2.632869157402243e+01
					1  'Sara'                   1.700000000000000e+01     2.632869157402243e+01
					1  'Wane'                   3.200000000000000e+01     2.632869157402243e+01
					2  'Bruce'                  5.000000000000000e+01     1.604992211819110e+01
					2  'Jane'                   5.000000000000000e+01     1.604992211819110e+01
					2  'Lee'                    8.800000000000000e+01     1.604992211819110e+01
					2  'Sara'                   5.500000000000000e+01     1.604992211819110e+01
					2  'Wane'                   4.200000000000000e+01     1.604992211819110e+01
					3  'Bruce'                  8.000000000000000e+01     2.085185843036539e+01
					3  'Jane'                   6.000000000000000e+01     2.085185843036539e+01
					3  'Lee'                    9.300000000000000e+01     2.085185843036539e+01
					3  'Sara'                   4.300000000000000e+01     2.085185843036539e+01
					3  'Wane'                   9.900000000000000e+01     2.085185843036539e+01

.. function:: STDDEV_SAMP( [ { DISTINCT | DISTINCTROW } | UNIQUE | ALL] expression )

	The **STDDEV_SAMP** function calculates the sample standard variance. Only one *expression* is specified as a parameter. If the **DISTINCT** or **UNIQUE** keyword is inserted before the expression, it calculates the sample standard variance after deleting duplicates; if a keyword is omitted or **ALL**, it calculates the sample standard variance for all values.

	:param expression: An expression that returns a numeric value
	:param ALL: Used to calculate the standard variance for all values. It is the default value.
	:param DISTINCT, UNIQUE: Used used to calculate the standard variance for the unique values without duplicates.
	:rtype: DOUBLE

	The return value is the same as the square root of it's sample variance (`VAR_SAMP Function <#syntax_syntax_operator_multi_var_3046>`_) and it is a **DOUBLE** type. If there are no rows that can be used for calculating a result, **NULL** is returned.

	The following are the formulas applied to the function.

	.. math:: STDDEV_SAMP = [ { 1 / (N-1) } * SUM( { xI - mean(x) }^2) ]^1/2

	The following example shows how to output the sample standard variance of all students for all subjects.

	.. code-block:: sql
	
		CREATE TABLE student (name VARCHAR(32), subjects_id INT, score DOUBLE);
		INSERT INTO student VALUES
		('Jane',1, 78),
		('Jane',2, 50),
		('Jane',3, 60),
		('Bruce', 1, 63),
		('Bruce', 2, 50),
		('Bruce', 3, 80),
		('Lee', 1, 85),
		('Lee', 2, 88),
		('Lee', 3, 93),
		('Wane', 1, 32),
		('Wane', 2, 42),
		('Wane', 3, 99),
		('Sara', 1, 17),
		('Sara', 2, 55),
		('Sara', 3, 43);
		 
		SELECT STDDEV_SAMP(score) FROM student;
		 
				stddev_samp(score)
		==========================
			 2.411480477888654e+01

	The following example shows how to output the sample standard variance of all students for all subjects.

	.. code-block:: sql
	
		SELECT subjects_id, name, score, STDDEV_SAMP(score) OVER(PARTITION BY subjects_id) std_samp FROM student ORDER BY subjects_id, name;
		 
		  subjects_id  name                                     score                  std_samp
		=======================================================================================
					1  'Bruce'                  6.300000000000000e+01     2.943637205907005e+01
					1  'Jane'                   7.800000000000000e+01     2.943637205907005e+01
					1  'Lee'                    8.500000000000000e+01     2.943637205907005e+01
					1  'Sara'                   1.700000000000000e+01     2.943637205907005e+01
					1  'Wane'                   3.200000000000000e+01     2.943637205907005e+01
					2  'Bruce'                  5.000000000000000e+01     1.794435844492636e+01
					2  'Jane'                   5.000000000000000e+01     1.794435844492636e+01
					2  'Lee'                    8.800000000000000e+01     1.794435844492636e+01
					2  'Sara'                   5.500000000000000e+01     1.794435844492636e+01
					2  'Wane'                   4.200000000000000e+01     1.794435844492636e+01
					3  'Bruce'                  8.000000000000000e+01     2.331308645374953e+01
					3  'Jane'                   6.000000000000000e+01     2.331308645374953e+01
					3  'Lee'                    9.300000000000000e+01     2.331308645374953e+01
					3  'Sara'                   4.300000000000000e+01     2.331308645374953e+01
					3  'Wane'                   9.900000000000000e+01     2.331308645374953e+01

.. function:: SUM ( [ { DISTINCT | DISTINCTROW } | UNIQUE | ALL ] expression )

	The **SUM** function returns the sum of expressions of all rows. Only one *expression* is specified as a parameter. You can get the sum without duplicates by inserting the **DISTINCT** or **UNIQUE** keyword in front of the expression, or get the sum of all values by omitting the keyword or by using **ALL**.

	:param expression: Specifies an expression that returns a numeric value.
	:param ALL: Gets the sum for all data (default).
	:param DISTINCT, UNIQUE: Gets the sum of unique values without duplicates
	:rtype: same type as that the expression

	The following is an example that outputs the top 10 countries and the total number of gold medals based on the sum of gold medals won in the Olympic Games in *demodb*.

	.. code-block:: sql
		
		SELECT nation_code, SUM(gold) FROM participant GROUP BY nation_code
		ORDER BY SUM(gold) DESC
		FOR ORDERBY_NUM() BETWEEN 1 AND 10 ;
		 
		=== <Result of SELECT Command in Line 1> ===
		 
		  nation_code             sum(gold)
		===================================
		  'USA'                         190
		  'CHN'                          97
		  'RUS'                          85
		  'GER'                          79
		  'URS'                          55
		  'FRA'                          53
		  'AUS'                          52
		  'ITA'                          48
		  'KOR'                          48
		  'EUN'                          45

	The following example shows how to output the number of gold medals by year and the average sum of the accumulated gold medals to the year acquired by the country whose nation_code code starts with 'AU' in *demodb*.

	.. code-block:: sql
	
		SELECT host_year, nation_code, gold,
		SUM(gold) OVER (PARTITION BY nation_code ORDER BY host_year) sum_gold
		FROM participant WHERE nation_code LIKE 'AU%';
		 
			host_year  nation_code                  gold     sum_gold
		=============================================================
				 1988  'AUS'                           3            3
				 1992  'AUS'                           7           10
				 1996  'AUS'                           9           19
				 2000  'AUS'                          16           35
				 2004  'AUS'                          17           52
				 1988  'AUT'                           1            1
				 1992  'AUT'                           0            1
				 1996  'AUT'                           0            1
				 2000  'AUT'                           2            3
				 2004  'AUT'                           2            5

	The following example is removing the "ORDER BY host_year" clause under the **OVER** analysis clause from the above example. The avg_gold value is the average of gold medals for all years, so the value is identical for every year by nation_code.

	.. code-block:: sql
	
		SELECT host_year, nation_code, gold, SUM(gold) OVER (PARTITION BY nation_code) sum_gold
		FROM participant WHERE nation_code LIKE 'AU%';
			host_year  nation_code                  gold     sum_gold
		=============================================================
				 2004  'AUS'                          17           52
				 2000  'AUS'                          16           52
				 1996  'AUS'                           9           52
				 1992  'AUS'                           7           52
				 1988  'AUS'                           3           52
				 2004  'AUT'                           2            5
				 2000  'AUT'                           2            5
				 1996  'AUT'                           0            5
				 1992  'AUT'                           0            5
				 1988  'AUT'                           1            5

.. function:: VAR_POP( [ DISTINCT | UNIQUE | ALL] expression )
.. function:: VARIANCE( [ DISTINCT | UNIQUE | ALL] expression )

	The functions **VARPOP** and **VARIANCE** are used interchangeably and they return a variance of expression values for all rows. Only one *expression* is specified as a parameter. If the **DISTINCT** or **UNIQUE** keyword is inserted before the expression, they calculate the population variance after deleting duplicates; if the keyword is omitted or **ALL**, they calculate the sample population variance for all values.

	:param expression: Specifies an expression that returns a numeric value.
	:param ALL: Gets the variance for all values (default).
	:param DISTINCT, UNIQUE: Gets the variance of unique values without duplicates.
	:rtype: DOUBLE

	The return value is a **DOUBLE** type. If there are no rows that can be used for calculating a result, **NULL** will be returned.

	The following is a formula that is applied to the function.

	.. math:: VAR_POP = (1/N) * SUM( { xI - AVG(x) }^2 )

	.. note:: In CUBRID 2008 R3.1 or earlier, the **VARIANCE** function worked the same as the :func:`VAR_SAMP`.

	The following example shows how to output the population variance of all students for all subjects

	.. code-block:: sql
	
		CREATE TABLE student (name VARCHAR(32), subjects_id INT, score DOUBLE);
		INSERT INTO student VALUES
		('Jane',1, 78),
		('Jane',2, 50),
		('Jane',3, 60),
		('Bruce', 1, 63),
		('Bruce', 2, 50),
		('Bruce', 3, 80),
		('Lee', 1, 85),
		('Lee', 2, 88),
		('Lee', 3, 93),
		('Wane', 1, 32),
		('Wane', 2, 42),
		('Wane', 3, 99),
		('Sara', 1, 17),
		('Sara', 2, 55),
		('Sara', 3, 43);
		 
		SELECT VAR_POP(score) FROM student;
		 
					var_pop(score)
		==========================
			 5.427555555555550e+02

	The following example shows how to output the score and population variance of all students by subject (subjects_id).

	.. code-block:: sql
	
		SELECT subjects_id, name, score, VAR_POP(score) OVER(PARTITION BY subjects_id) v_pop
		FROM student ORDER BY subjects_id, name;
		 
		  subjects_id  name                                     score                     v_pop
		=======================================================================================
					1  'Bruce'                  6.300000000000000e+01     6.931999999999998e+02
					1  'Jane'                   7.800000000000000e+01     6.931999999999998e+02
					1  'Lee'                    8.500000000000000e+01     6.931999999999998e+02
					1  'Sara'                   1.700000000000000e+01     6.931999999999998e+02
					1  'Wane'                   3.200000000000000e+01     6.931999999999998e+02
					2  'Bruce'                  5.000000000000000e+01     2.575999999999999e+02
					2  'Jane'                   5.000000000000000e+01     2.575999999999999e+02
					2  'Lee'                    8.800000000000000e+01     2.575999999999999e+02
					2  'Sara'                   5.500000000000000e+01     2.575999999999999e+02
					2  'Wane'                   4.200000000000000e+01     2.575999999999999e+02
					3  'Bruce'                  8.000000000000000e+01     4.348000000000002e+02
					3  'Jane'                   6.000000000000000e+01     4.348000000000002e+02
					3  'Lee'                    9.300000000000000e+01     4.348000000000002e+02
					3  'Sara'                   4.300000000000000e+01     4.348000000000002e+02
					3  'Wane'                   9.900000000000000e+01     4.348000000000002e+02

.. function:: VAR_SAMP( [ DISTINCT | UNIQUE | ALL] expression )

	The **VAR_SAMP** function returns the sample variance. The denominator is the number of all rows - 1. Only one *expression* is specified as a parameter. If the **DISTINCT** or **UNIQUE** keyword is inserted before the expression, it calculates the sample variance after deleting duplicates and if the keyword is omitted or **ALL**, it calculates the sample variance for all values.

	:param expression: Specifies one expression to return the numeric.
	:param ALL: Is used to calculate the sample variance of unique values without duplicates. It is the default value.
	:param DISTINCT, UNIQUE: Is used to calculate the sample variance for the unique values without duplicates.
	:rtype: DOUBLE

	The return value is a **DOUBLE** type. If there are no rows that can be used for calculating a result, **NULL** is returned.

	The following are the formulas applied to the function.

	.. math:: VAR_SAMP = { 1 / (N-1) } * SUM( { xI - mean(x) }^2 )

	The following example shows how to output the sample variance of all students for all subjects.

	.. code-block:: sql
	
		CREATE TABLE student (name VARCHAR(32), subjects_id INT, score DOUBLE);
		INSERT INTO student VALUES
		('Jane',1, 78),
		('Jane',2, 50),
		('Jane',3, 60),
		('Bruce', 1, 63),
		('Bruce', 2, 50),
		('Bruce', 3, 80),
		('Lee', 1, 85),
		('Lee', 2, 88),
		('Lee', 3, 93),
		('Wane', 1, 32),
		('Wane', 2, 42),
		('Wane', 3, 99),
		('Sara', 1, 17),
		('Sara', 2, 55),
		('Sara', 3, 43);
		 
		SELECT VAR_SAMP(score) FROM student;
				   var_samp(score)
		==========================
			 5.815238095238092e+02

	The following example shows how to output the score and sample variance of all students by subject (subjects_id).

	.. code-block:: sql
	
		SELECT subjects_id, name, score, VAR_SAMP(score) OVER(PARTITION BY subjects_id) v_samp
		FROM student ORDER BY subjects_id, name;
		 
		  subjects_id  name                                     score                    v_samp
		=======================================================================================
					1  'Bruce'                  6.300000000000000e+01     8.665000000000000e+02
					1  'Jane'                   7.800000000000000e+01     8.665000000000000e+02
					1  'Lee'                    8.500000000000000e+01     8.665000000000000e+02
					1  'Sara'                   1.700000000000000e+01     8.665000000000000e+02
					1  'Wane'                   3.200000000000000e+01     8.665000000000000e+02
					2  'Bruce'                  5.000000000000000e+01     3.220000000000000e+02
					2  'Jane'                   5.000000000000000e+01     3.220000000000000e+02
					2  'Lee'                    8.800000000000000e+01     3.220000000000000e+02
					2  'Sara'                   5.500000000000000e+01     3.220000000000000e+02
					2  'Wane'                   4.200000000000000e+01     3.220000000000000e+02
					3  'Bruce'                  8.000000000000000e+01     5.435000000000000e+02
					3  'Jane'                   6.000000000000000e+01     5.435000000000000e+02
					3  'Lee'                    9.300000000000000e+01     5.435000000000000e+02
					3  'Sara'                   4.300000000000000e+01     5.435000000000000e+02
					3  'Wane'                   9.900000000000000e+01     5.435000000000000e+02