# TransMilenio Data


Trasmilenio manages the integrated transport system in Bogota, which
includes local buses, BRT, and cable-cars. This repository was created
as part of the Mini-Hack taking place in ITS at the University of Leeds
on May 8th, 2025.

## Download the data

A sample of the data is available in the releases of this repository
[here](https://github.com/juanfonsecaLS1/TM_datos/releases/tag/v0). The
full dataset can be obtained directly from the data portal of
Transmilenio [here](https://datosabiertos-transmilenio.hub.arcgis.com/).

## Pre-requisites

``` r
options(repos = c(CRAN = "https://cloud.r-project.org"))
if (!require("remotes")) install.packages("remotes")
pkgs = c(
    "sf",
    "tidyverse",
    "tmap"
)
remotes::install_cran(pkgs)
sapply(pkgs, require, character.only = TRUE)
```

           sf tidyverse      tmap 
         TRUE      TRUE      TRUE 

## Extracting the data

Listing the files

``` r
zip_files <- list.files(pattern = "\\.zip")
zip_files
```

    [1] "consolidado_2024_trunk.zip"    "consolidado_2024_zonal.zip"   
    [3] "salidas20250409.zip"           "validacionTroncal20250409.zip"

Extracting the files

``` r
sapply(zip_files,\(x){
  # dir.create(gsub(pattern = "\\.zip",replacement = "",x = basename(x)))}
  unzip(x,exdir = gsub(pattern = "\\.zip",replacement = "",x = basename(x)))
  }
  )
```

                   consolidado_2024_trunk.zip 
    "consolidado_2024_trunk/troncal_2024.csv" 
                   consolidado_2024_zonal.zip 
      "consolidado_2024_zonal/zonal_2024.csv" 
                          salidas20250409.zip 
               "salidas20250409/20250409.csv" 
                validacionTroncal20250409.zip 
     "validacionTroncal20250409/20250409.csv" 

## Reading the data

``` r
boardingTrunk <- read_csv("consolidado_2024_trunk/troncal_2024.csv")
boardingZonal <- read_csv("consolidado_2024_zonal/zonal_2024.csv")
tapindaily <- read_csv("validacionTroncal20250409/20250409.csv")
tripendsdaily <- read_csv("salidas20250409/20250409.csv")
```

## Exploring Boarding data

These are hourly boardings by station for both BRT and Zonal buses

``` r
str(boardingTrunk)
```

    spc_tbl_ [1,628,608 × 7] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
     $ Estacion_Parada: chr [1:1628608] "(11359) 226A00-Pedagogica" "(11359) 226A00-Pedagogica" "(11359) 226A00-Pedagogica" "(11359) 226A00-Pedagogica" ...
     $ latitud        : num [1:1628608] 4.66 4.66 4.66 4.66 4.66 ...
     $ longitud       : num [1:1628608] -74.1 -74.1 -74.1 -74.1 -74.1 ...
     $ fecha          : Date[1:1628608], format: "2024-01-10" "2024-01-11" ...
     $ hora           : num [1:1628608] 19 19 17 19 20 14 18 19 20 11 ...
     $ Sistema        : chr [1:1628608] "DUAL" "DUAL" "DUAL" "DUAL" ...
     $ validaciones   : num [1:1628608] 1 7 9 5 1 1 17 5 5 2 ...
     - attr(*, "spec")=
      .. cols(
      ..   Estacion_Parada = col_character(),
      ..   latitud = col_double(),
      ..   longitud = col_double(),
      ..   fecha = col_date(format = ""),
      ..   hora = col_double(),
      ..   Sistema = col_character(),
      ..   validaciones = col_double()
      .. )
     - attr(*, "problems")=<externalptr> 

``` r
str(boardingZonal)
```

    spc_tbl_ [34,016,995 × 7] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
     $ Estacion_Parada: chr [1:34016995] "(49841) 080A01_TM|080A01_Br. La Liberia" "(49841) 080A01_TM|080A01_Br. La Liberia" "(49841) 080A01_TM|080A01_Br. La Liberia" "(49841) 080A01_TM|080A01_Br. La Liberia" ...
     $ latitud        : num [1:34016995] 4.74 4.74 4.74 4.74 4.74 ...
     $ longitud       : num [1:34016995] -74 -74 -74 -74 -74 ...
     $ fecha          : Date[1:34016995], format: "2024-01-01" "2024-01-01" ...
     $ hora           : num [1:34016995] 10 11 12 13 14 15 16 17 18 19 ...
     $ Sistema        : chr [1:34016995] "ZONAL" "ZONAL" "ZONAL" "ZONAL" ...
     $ validaciones   : num [1:34016995] 3 5 1 5 10 12 12 6 24 5 ...
     - attr(*, "spec")=
      .. cols(
      ..   Estacion_Parada = col_character(),
      ..   latitud = col_double(),
      ..   longitud = col_double(),
      ..   fecha = col_date(format = ""),
      ..   hora = col_double(),
      ..   Sistema = col_character(),
      ..   validaciones = col_double()
      .. )
     - attr(*, "problems")=<externalptr> 

A translation of the column names:

| Col name        | Translation           |
|-----------------|-----------------------|
| Estacion_Parada | stop                  |
| fecha           | date                  |
| hora            | hour                  |
| Sistema         | system                |
| validaciones    | validations (tap-ins) |

## Exploring Tap-In data

This dataset is very detailed, containing each individual trip made
using the BRT system

``` r
str(tapindaily)
```

    spc_tbl_ [1,982,649 × 22] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
     $ Acceso_Estacion           : chr [1:1982649] "(08) PLATAFORMA 1 ALIMENTACIÓN - CASA BLANCA/AV VILLAVICENCIO/PATIO BONITO/ROMA" "(08) PLATAFORMA 1 ALIMENTACIÓN - CASA BLANCA/AV VILLAVICENCIO/PATIO BONITO/ROMA" "(08) PLATAFORMA 1 ALIMENTACIÓN - CASA BLANCA/AV VILLAVICENCIO/PATIO BONITO/ROMA" "(08) PLATAFORMA 1 ALIMENTACIÓN - CASA BLANCA/AV VILLAVICENCIO/PATIO BONITO/ROMA" ...
     $ Day_Group_Type            : chr [1:1982649] "Dia 1" "Dia 1" "Dia 1" "Dia 1" ...
     $ Dispositivo               : chr [1:1982649] "040000167" "040000164" "040000169" "040000164" ...
     $ Emisor                    : chr [1:1982649] "(3101000) Bogota Card(Citizen)" "(3101000) Bogota Card(Citizen)" "(3101000) Bogota Card(Citizen)" "(3101000) Bogota Card(Citizen)" ...
     $ Estacion_Parada           : chr [1:1982649] "(05000) Portal Américas" "(05000) Portal Américas" "(05000) Portal Américas" "(05000) Portal Américas" ...
     $ Fase                      : chr [1:1982649] "Fase 3" "Fase 3" "Fase 3" "Fase 3" ...
     $ Fecha_Clearing            : Date[1:1982649], format: "2025-04-09" "2025-04-09" ...
     $ Fecha_Transaccion         : POSIXct[1:1982649], format: "2025-04-09 03:50:21" "2025-04-09 03:50:22" ...
     $ Hora_Pico_SN              : chr [1:1982649] "Peak Time" "Peak Time" "Peak Time" "Peak Time" ...
     $ ID_Vehiculo               : logi [1:1982649] NA NA NA NA NA NA ...
     $ Linea                     : chr [1:1982649] "(31) Zona F Av. Américas" "(31) Zona F Av. Américas" "(31) Zona F Av. Américas" "(31) Zona F Av. Américas" ...
     $ Nombre_Perfil             : chr [1:1982649] "(001) Anonymous" "(001) Adulto" "(001) Adulto" "(002) Adulto Mayor" ...
     $ Numero_Tarjeta            : chr [1:1982649] "cd5907b25fbc99d2ebb79df6a50c29913fd6a6ff7af6eebcceaac62aa1965cfc" "6017762340927c4b3a53e10ebcd95ab644f0c9cc7448f263f8ee9083946ef5d4" "84085f1f97f899a8876d2d7ac3f5a38b6cc842140c8ccf1b57daac6f410aac6b" "eea960eceb9218cd6240f60c40f611e8e41d5dded1a8d241dbbd02879414d158" ...
     $ Operador                  : chr [1:1982649] "(201) Trunk agency" "(201) Trunk agency" "(201) Trunk agency" "(201) Trunk agency" ...
     $ Ruta                      : logi [1:1982649] NA NA NA NA NA NA ...
     $ Saldo_Despues_Transaccion : num [1:1982649] 25100 1050 10550 15180 32600 ...
     $ Saldo_Previo_a_Transaccion: num [1:1982649] 28300 4250 13750 18380 35800 ...
     $ Sistema                   : chr [1:1982649] "TRONCAL" "TRONCAL" "TRONCAL" "TRONCAL" ...
     $ Tipo_Tarifa               : num [1:1982649] 10 10 10 10 10 10 10 10 10 10 ...
     $ Tipo_Tarjeta              : chr [1:1982649] "tullave Básica" "tullave Plus" "tullave Plus" "tullave Plus" ...
     $ Tipo_Vehiculo             : logi [1:1982649] NA NA NA NA NA NA ...
     $ Valor                     : num [1:1982649] 3200 3200 3200 3200 3200 3200 3200 3200 3200 3200 ...
     - attr(*, "spec")=
      .. cols(
      ..   Acceso_Estacion = col_character(),
      ..   Day_Group_Type = col_character(),
      ..   Dispositivo = col_character(),
      ..   Emisor = col_character(),
      ..   Estacion_Parada = col_character(),
      ..   Fase = col_character(),
      ..   Fecha_Clearing = col_date(format = ""),
      ..   Fecha_Transaccion = col_datetime(format = ""),
      ..   Hora_Pico_SN = col_character(),
      ..   ID_Vehiculo = col_logical(),
      ..   Linea = col_character(),
      ..   Nombre_Perfil = col_character(),
      ..   Numero_Tarjeta = col_character(),
      ..   Operador = col_character(),
      ..   Ruta = col_logical(),
      ..   Saldo_Despues_Transaccion = col_double(),
      ..   Saldo_Previo_a_Transaccion = col_double(),
      ..   Sistema = col_character(),
      ..   Tipo_Tarifa = col_number(),
      ..   Tipo_Tarjeta = col_character(),
      ..   Tipo_Vehiculo = col_logical(),
      ..   Valor = col_double()
      .. )
     - attr(*, "problems")=<externalptr> 

A translation of the column names:

| Col name                   | Translation                       |
|----------------------------|-----------------------------------|
| Acceso_Estacion            | access id                         |
| Day_Group_Type             | day type                          |
| Dispositivo                | device (where tapped-in )         |
| Emisor                     | card issuer                       |
| Estacion_Parada            | station/stop where tapped-in      |
| Fase                       | BRT phase                         |
| Fecha_Clearing             | date of transaction               |
| Fecha_Transaccion          | transaction’s timestamp           |
| Hora_Pico_SN               | logical if peak hour              |
| ID_Vehiculo                | vehicle id for zonal buses        |
| Linea                      | line                              |
| Nombre_Perfil              | profile name                      |
| Numero_Tarjeta             | card id                           |
| Operador                   | vehicle operator                  |
| Ruta                       | service id                        |
| Saldo_Despues_Transaccion  | balance after tap-in              |
| Saldo_Previo_a_Transaccion | balance before tap-in             |
| Sistema                    | system either Trunk(BRT) or Zonal |
| Tipo_Tarifa                | fare type                         |
| Tipo_Tarjeta               | card type                         |
| Tipo_Vehiculo              | vehicle type                      |
| Valor                      | fare cost                         |

Can you figure out and visualise:

- the daily trips of a card with a few records?

- travel patterns of 62+ yr users?

- …

## Trip Ends

This dataset contains the number of daily trip-ends (people boarding or
leaving the system)

``` r
str(tripendsdaily)
```

    spc_tbl_ [127,256 × 8] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
     $ Fecha_Transaccion: Date[1:127256], format: "2025-04-09" "2025-04-09" ...
     $ Tiempo           : 'hms' num [1:127256] 00:00:00 00:00:00 00:00:00 00:00:00 ...
      ..- attr(*, "units")= chr "secs"
     $ Linea            : chr [1:127256] "(36)Zona A Caracas" "(36)Zona A Caracas" "(33)Zona B AutoNorte" "(33)Zona B AutoNorte" ...
     $ Estacion         : chr [1:127256] "(09115)Calle 34 - Fondo Nacional de Garantias" "(09115)Calle 34 - Fondo Nacional de Garantias" "(02202)Calle 127" "(02202)Calle 127" ...
     $ Acceso_Estacion  : chr [1:127256] "(AA) Acceso Norte" "(AA) Acceso Norte" "(AA) Acceso Norte" "(AA) Acceso Norte" ...
     $ Dispositivo      : chr [1:127256] "010000841" "010000842" "010000848" "010000849" ...
     $ Entradas_E       : num [1:127256] 0 0 0 0 0 0 0 0 0 0 ...
     $ Salidas_S        : num [1:127256] 2 2 0 0 0 0 0 0 4 1 ...
     - attr(*, "spec")=
      .. cols(
      ..   Fecha_Transaccion = col_date(format = ""),
      ..   Tiempo = col_time(format = ""),
      ..   Linea = col_character(),
      ..   Estacion = col_character(),
      ..   Acceso_Estacion = col_character(),
      ..   Dispositivo = col_character(),
      ..   Entradas_E = col_double(),
      ..   Salidas_S = col_double()
      .. )
     - attr(*, "problems")=<externalptr> 

A translation of the metadata:

A translation of the column names:

| Col name          | Translation  |
|-------------------|--------------|
| Fecha_Transaccion | date         |
| Tiempo            | time         |
| Linea             | line (zone)  |
| Estacion          | stop/station |
| Acceso_Estacion   | access id    |
| Dispositivo       | device       |
| Entradas_E        | entry_counts |
| Salidas_S         | exit_counts  |

Can you estimate:

- users not paying the fare?

- travel patterns across the city?

- …
