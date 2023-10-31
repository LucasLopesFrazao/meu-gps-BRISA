# Configuração e Execução do OSRM

Este guia explica como instalar e executar a ferramenta OSRM (OpenStreetMap Routing Machine) utilizando Docker.

## Pré-requisitos

### Docker

Instalação: Para começar, é necessário ter o Docker instalado em sua máquina.

[Clique aqui](https://docs.docker.com/engine/install/ubuntu/) para obter as instruções de instalação do Docker para Ubuntu.

### Dados Geográficos
Download dos dados do OpenStreetMap, baixe os dados da região Centro-Oeste do Brasil a partir do [Geofabrik](https://download.geofabrik.de/south-america/brazil/centro-oeste-latest.osm.pbf):

## Configurando o OSRM

Para uma compreensão completa, consulte a [documentação oficial do OSRM](https://github.com/Project-OSRM/osrm-backend). Abaixo está um resumo dos passos necessários:

### Processamento dos Dados

Certifique-se de que o Docker esteja em execução e, em seguida, execute os seguintes comandos:

#### Extração:

```docker run -t -v "${PWD}:/data" ghcr.io/project-osrm/osrm-backend osrm-extract -p /opt/car.lua /data/centro-oeste-latest.osm.pbf || echo "osrm-extract failed"```

#### Particionamento e customização

```docker run -t -v "${PWD}:/data" ghcr.io/project-osrm/osrm-backend osrm-partition /data/centro-oeste-latest.osrm || echo "osrm-partition failed"```

```docker run -t -v "${PWD}:/data" ghcr.io/project-osrm/osrm-backend osrm-customize /data/centro-oeste-latest.osrm || echo "osrm-customize failed"```

#### Executando o Servidor OSRM

```docker run -t -i -p 5000:5000 -v "${PWD}:/data" ghcr.io/project-osrm/osrm-backend osrm-routed --algorithm mld /data/centro-oeste-latest.osrm```

### Bibliotecas Adicionais

Instale as bibliotecas que estão em `requirements.txt`

## Conclusão

Após seguir esses passos, o servidor OSRM estará em execução localmente na porta 5000. Você pode acessar através de http://localhost:5000.

# Como Interpretar e Utilizar a Resposta do Endpoint

Ao utilizar a ferramenta OSRM para obter rotas, é possível consultar diretamente um endpoint específico para obter os detalhes da rota.

## Realizando a Consulta

Faça uma solicitação GET para o seguinte endpoint:

```http://127.0.0.1:5000/route/v1/driving/-48.043619,-16.000091;-48.043619,-15.842141```

A resposta deve ser semelhante a:

```
{
  "code": "Ok",
  "routes": [
    {
      "geometry": "f|s`BpufdHnEtC}E|ItX|RmJ`Pei@m@_uDioEqLzRecB~sA_v@xb@unJnwBgjAgb@ePr@w_@se@_Pwf@{qAdj@oWuj@vGgCvBvE",
      "legs": [
        {
          "steps": [],
          "summary": "",
          "weight": 1698.7,
          "duration": 1698.7,
          "distance": 23945.3
        }
      ],
      "weight_name": "routability",
      "weight": 1698.7,
      "duration": 1698.7,
      "distance": 23945.3
    }
  ],
  "waypoints": [
    {
      "hint": "BGoQgApqEIAXAAAAAAAAAHQAAAB9AAAAW4IbQQAAAACHxkFC8otRQhcAAAAAAAAAdAAAAH0AAADCAAAA6OUi_bfdC_-d6SL9pdsL_wEALwJgC8Vo",
      "distance": 117.30249163,
      "name": "",
      "location": [
        -48.044568,
        -15.999561
      ]
    },
    {
      "hint": "v6UTgP___389AAAAQQAAAAAAAABZAAAAaPGjQiO-qUAAAAAAN6e4Qj0AAABBAAAAAAAAAFkAAADCAAAAm-ki_ahEDv-d6SL9o0QO_wAAXw9gC8Vo",
      "distance": 0.593319936,
      "name": "Avenida Arniqueira",
      "location": [
        -48.043621,
        -15.842136
      ]
    }
  ]
}
```
## Entendendo a Resposta

O campo de geometry na resposta contém uma string que representa o caminho da rota em um formato compacto. Para converter essa string em coordenadas legíveis, é necessário decodificar usando a função `polyline`.

```
import polyline

decoded_path = polyline.decode('f|s`BpufdHnEtC...')

print(decoded_path)
```

Agora, você terá um array de arrays com as coordenadas necessárias para desenhar o caminho:

```
[(-15.99956, -48.04457),
 (-16.0006, -48.04532),
 (-15.99949, -48.04707),
 (-16.0036, -48.05026),
 (-16.00177, -48.05299),
 (-15.99502, -48.05276),
 (-15.9659, -48.01943),
 (-15.96373, -48.02261),
 (-15.9477, -48.03621),
 (-15.9389, -48.04194),
 (-15.88007, -48.06122),
 (-15.86803, -48.05558),
 (-15.86528, -48.05584),
 (-15.86004, -48.04966),
 (-15.85732, -48.0433),
 (-15.84406, -48.05021),
 (-15.84014, -48.04322),
 (-15.84154, -48.04254),
 (-15.84214, -48.04362)]
```

Estas coordenadas podem ser usadas em diversas aplicações, como desenhar o caminho em um mapa.