Podemos señalar que en la normalización de la data a través de la limpieza en power query la misma no sufrió cambios significativos de fondo pero si cambios esenciales de forma donde destaca principalmente el establecer los nombres de cada una de las columnas en las distintas tablas requeridas para el proyecto, además se evaluaron todas la columnas para que los formatos estuvieses acorde a los datos respectivamente, de igual forma se realizaron unión de tablas respondiendo a los requerimientos donde destaca la incorporación de columnas de DimGeography a la tabla de DimCustomer bajo el siguiente esquema: 
#"Consultas combinadas" = Table.NestedJoin(#"Columnas quitadas2", {"GeographyKey"}, DimGeography, {"GeographyKey"}, "DimGeography", JoinKind.LeftOuter), 
#"Se expandió DimGeography" = Table.ExpandTableColumn(#"Consultas combinadas", "DimGeography", {"City", "StateProvinceCode", "StateProvinceName"}, {"DimGeography.City", "DimGeography.StateProvinceCode", "DimGeography.StateProvinceName"}),
#"Columnas con nombre cambiado" = Table.RenameColumns(#"Se expandió DimGeography",{{"DimGeography.City", "City"}, {"DimGeography.StateProvinceCode", "StateProvinceCode"}, {"DimGeography.StateProvinceName", "StateProvinceName"}}).

En este mismo orden de ideas señalamos que se realizó la unión columnas dentro de la tabla DimCustomer de la siguiente forma:
#"Valor reemplazado" = Table.ReplaceValue(#"Columnas quitadas1",null," ",Replacer.ReplaceValue,{"CountryRegionCode"}),
#"Valor reemplazado1" = Table.ReplaceValue(#"Valor reemplazado",null," ",Replacer.ReplaceValue,{"CountryRegionCode_1", "CountryRegionCode_2", "CountryRegionCode_3", "CountryRegionCode_4", "CountryRegionCode_5"}),
#"Columna combinada insertada" = Table.AddColumn(#"Valor reemplazado1", "CountryCodeRegion", each Text.Combine({[CountryRegionCode], [CountryRegionCode_1], [CountryRegionCode_2], [CountryRegionCode_3], [CountryRegionCode_4], [CountryRegionCode_5]}, ""), type text),
#"Columnas quitadas2" = Table.RemoveColumns(#"Columna combinada insertada",{"CountryRegionCode", "CountryRegionCode_1", "CountryRegionCode_2", "CountryRegionCode_3", "CountryRegionCode_4", "CountryRegionCode_5"}).

	Posterior a la normalización inicial realizada desde power query se realizaron distintas acciones desde Power BI creando tablas y columnas calculadas directamente desde nuestro modelo relacional, tales como:
MedidasFinancieras
Variacion_Tiempo
Variacion_Tiempo_Utilidad


De la misma forma, en cuanto a la implementación de variables, medidas, columnas calculadas y otras que den respuesta a los objetivos y kpi´s que buscamos en nuestro proyecto y que responden a los requerimientos, a continuación señalo las medidas más destacadas:

TotalIngresos = SUM(FactInternetSales[SalesAmount]) (Nos permite medir el ingreso generado por las ventas)
TotalCost = SUM(FactInternetSales[Cost]) (Costo global generado)
COGS = SUM('FactInternetSales'[TotalProductCost]) 
Gastos_Operativos = [TotalIngresos] * 0.20 (gastos operativos generados)


CantidadVendida = SUM(FactInternetSales[OrderQuantity])(total de cantidad vendida)
Utilidad_Bruta = [TotalIngresos] - [COGS]
Utilidad_Neta = [Utilidad_Bruta] - [Gastos_Operativos]


Por otra parte en la generación de columnas calculadas podemos encontrar la siguiente:


Cost = FactInternetSales[TotalProductCost] + FactInternetSales[Freight] + FactInternetSales[DiscountAmount]


Finalmente podemos destacar que para este proyecto no ejecutamos una tabla calendario ya que nos pareció que no hacía falta la misma, sin embargo en esta fase se generaron dos tablas respecto a la variación del tiempo relacionándola directamente con DimDate, ejecutando el siguiente código:


Variacion_Tiempo = 
UNION(
    SELECTCOLUMNS(
        ADDCOLUMNS(
            CROSSJOIN(
                VALUES('DimDate'[CalendarYear]),
                ROW("Período", "PeríodoActual")
            ),
            "Valor", [PeriodoActual]
        ),
        "Año", 'DimDate'[CalendarYear],
        "Período", [Período],
        "Valor", [Valor]
    ),
    SELECTCOLUMNS(
        ADDCOLUMNS(
            CROSSJOIN(
                VALUES('DimDate'[CalendarYear]),
                ROW("Período", "Período Anterior")
            ),
            "Valor", [Ventas_Periodo_Anterior]
        ),
        "Año", 'DimDate'[CalendarYear],
        "Período", [Período],
        "Valor", [Valor]
    ),
    SELECTCOLUMNS(
        ADDCOLUMNS(
            CROSSJOIN(
                VALUES('DimDate'[CalendarYear]),
                ROW("Período", "Variación")
            ),
            "Valor", [Variacion]
        ),
        "Año", 'DimDate'[CalendarYear],
        "Período", [Período],
        "Valor", [Valor]
    ),
    SELECTCOLUMNS(
        ADDCOLUMNS(
            CROSSJOIN(
                VALUES('DimDate'[CalendarYear]),
                ROW("Período", "Variación Porcentual")
            ),
            "Valor", [Variacion_Porcentual]
        ),
        "Año", 'DimDate'[CalendarYear],
        "Período", [Período],
        "Valor", [Valor]
    )
).
