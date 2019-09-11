### 1.PostgreSQL实现一个月的连续日期 ###
    select to_char(generate_series('2019-08-03'::date,current_date,'1 day'),'yyyy-mm-dd')

### 2.PostgreSQL实现获取一个月的数据，若哪天不存在数据，填充0 ###
    SELECT A
    	.TIME,
    	COALESCE ( b.bonus_value, 0 ) AS "bonus_value",
    	COALESCE ( b.bonus_type, NULL ) AS "bonus_type" 
    FROM
    	(
    	SELECT
    		to_char( b, 'YYYY-MM-DD' ) AS TIME 
    	FROM
    		generate_series ( to_timestamp( '2019-08-04', 'YYYY-MM-DD hh24:mi:ss' ), to_timestamp( '2019-09-04', 'YYYY-MM-DD hh24:mi:ss' ), '1 days' ) AS b 
    	GROUP BY
    	TIME 
    	ORDER BY
    	TIME ASC 
    	)
    	AS A FULL OUTER JOIN (
    	SELECT SUM
    		( aa.bonus_value ) AS bonus_value,
    		aa.oper_time,
    		aa.bonus_type 
    	FROM
    		(
    		SELECT
    			"id",
    			bonus_type,
    		CASE
    				
    				WHEN oper_type = 1 THEN
    				bonus_value ELSE - bonus_value 
    			END AS bonus_value,
    			to_char( oper_time, 'yyyy-mm-dd' ) AS oper_time 
    		FROM
    			bonus_log 
    		WHERE
    			user_id = '00000016' 
    			AND bonus_type = 'BONUS_GOLD' 
    			AND oper_type = 0 
    		ORDER BY
    		to_char( oper_time, 'yyyy-mm-dd' )) AS aa 
    	GROUP BY
    		aa.oper_time,
    		aa.bonus_type 
    	) AS b ON A.TIME = b.oper_time WHERE A.TIME NOTNULL
    ORDER BY
    	A.TIME ASC;