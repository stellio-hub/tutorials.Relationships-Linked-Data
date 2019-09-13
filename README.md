[![FIWARE Banner](https://fiware.github.io/tutorials.Relationships-Linked-Data/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Relationships-Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
[![NGSI LD](https://img.shields.io/badge/NGSI-linked_data-red.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.01.01_60/gs_CIM009v010101p.pdf)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

This tutorial discusses relationships between linked data entities and how the concepts of **JSON-LD** and **NGSI-LD**
can be used to interrogate entities and navigate from one entity to another. The tutorial discusses a series of simple
linked-data data models based around the supermarket chain’s store finder application, and demonstrates how to design
models holding one-to-one, one-to-many and many-to-many relationships. This **NGSI-LD** tutorial is a direct analogue to
the earlier _Understanding Entities and Relationships_ tutorial (which was based on the **NGSI v2** interface). The
differences in relationships created using **NSGI v2** and **NGSI-LD** are highlighted and discussed in detail.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://fiware.github.io/tutorials.Relationships-Linked-Data/)

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/125db8d3a1ea3dab8e3f)

# Relationships in Linked Data

All NGSI data entity attributes can be divided into one of two types.

-   _Property_ attributes
-   _Relationship_ attributes

For each entity, the _Property_ attributes (including various subtypes such as _GeoProperty_ , _TemporalProperty_ and
time values) define the current state something in the real world. As the state of the entity changes the `value` of
each _Property_ is updated to align with the last real world reading of the the attribute. All _Property_ attributes
relate to the state of a single entity.

_Relationship_ attributes correspond to the interactions **between** entities (which are expected to change over time).
They effectively provide the graph linking the nodes of the data entities together. Each _Relationship_ attribute holds
an `object` in the form of a URN - effectively a pointer to another object. _Relationship_ attributes do not hold data
themselves.

Both properties and relationships may in turn have a linked embedded structure (of _properties-of-properties_ or
_properties-of-relationships or relationships-of-properties_ or _relationships-of-relationships_ etc.) which lead a full
complex knowledge graph.

## Designing Data Models using JSON-LD

In order for computers to be able to navigate linked data structures, a full `@context` must be defined and accessible.
We can do this by reviewing and updating the existing data models from the NGSI v2
[Entity Relationships](https://github.com/FIWARE/tutorials.Entity-Relationships) tutorial.

### Revision: Data Models for a Stock management system as defined using NGSI-v2

As a reminder, four types of entity were created in the NGSI v2 stock management system. The relationship between the
four NGSI v2 entity models was defined as shown below:

![](https://jason-fox.github.io/tutorials.Relationships-Linked-Data/img/entities-v2.png)

-   A **Store** is a real world bricks and mortar building. **Store** entities have properties such as:
    -   A name of the store e.g. "Checkpoint Markt"
    -   An address "Friedrichstraße 44, 10969 Kreuzberg, Berlin"
    -   A phyiscal location e.g. _52.5075 N, 13.3903 E_
-   A shelf is a real world device to hold objects which we wish to sell. Each **Shelf** entity has properties such as:
    -   A name of the shelf e.g. "Wall Unit"
    -   A phyiscal location e.g. _52.5075 N, 13.3903 E_
    -   A maximum capacity
    -   A relationship to the store `refStore` in which the shelf is present
-   A **Product** is defined as something that we sell - it is conceptural object. **Product** entities have properties
    such as:
    -   A name of the product e.g. "Vodka"
    -   A price e.g. 13.99 Euros
    -   A size e.g. Small
-   An inventory item is another conceptural entity, used to assocate products, stores, shelves and physical objects.
    **Inventory Item** entities has properties such as:
    -   An relationship `refProduct` to the product being sold
    -   An relationship `refStore` to the store in which the product is being sold
    -   An relationship `refShelf` to the shelf where the product is being displayed
    -   A stock count of the quantity of the product available in the warehouse
    -   A stock count of the quantity of the product available on the shelf.

In NGSI v2 relationship attributes are just standard properties attributes. By convention NGSI v2 relationship
attributes are given names starting `ref` and are defined using the `type="Relationship"`. However, this is merely
convention and may not be followed in all cases. There is no infallible mechanism to detect which attributes are
association between entities.

### Data Models for a Stock management system defined using NGSI-LD

The richer JSON-LD description language is able to define NSGI-LD entities by linking entities directly as shown below.

![](https://jason-fox.github.io/tutorials.Relationships-Linked-Data/img/entities-ld.png)

- A full Human readable definition of this data model can be found [online](https://fiware.github.io/tutorials.Step-by-Step/schema).
- The machine readable JSON-LD defintion for t can be found at [`https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld`](https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld) - this file will be used to provide the `@context` to power our NGSI-LD data entities.

Four models have been created for the NGSI-LD stock management system.

-   The [**Store** model](https://fiware.github.io/tutorials.Step-by-Step/schema/Store/) is now based on and extends the FIWARE [**Building** model](https://fiware-datamodels.readthedocs.io/en/latest/Building/Building/doc/spec/index.html). This ensures that it offers standard properties for
    `name`, `address` and category.
    -   A Building will hold `furniture` this is a 1-many unidirectional relationship from Building to Shelf
-   The [**Shelf** model](https://fiware.github.io/tutorials.Step-by-Step/schema/Shelf/) is a custom data model defined for the tutorial
    -   Each **Shelf** is `locatedIn` a **Building**. This is a 1-1 unidirectional relationship from Shelf to Building.
        It is the reciprical relationship to `furniture` defined above.
    -   A **Shelf** is `installedBy` a **Person** - this is a unidirectional 1-1 relationship. A shelf knows who
        installed it, but it is this knowledge is not part of the Person entity itself.
    -   A **Shelf** `stocks` a given **Product**. This is another unidirectional 1-1 relationship, and again it is not
        recipricated. A **Product** does not know which **Shelf** it is to be found on.
-   A [**StockOrder** model]](https://fiware.github.io/tutorials.Step-by-Step/schema/StockOrder/) replaces the  **Inventory Item** bridge table defined for NGSI v2 :
    -   A **StockOrder** is `requestedBy` a **Person** - this is a unidirectional 1-1 relationship.
    -   A **StockOrder** is `requestedFor` a **Building** - this is a unidirectional 1-1 relationship.
    -   A **StockOrder** is a request for a specific `orderedProduct` - this unidirectional 1-1 relationship.
-   The [**Product** model](https://fiware.github.io/tutorials.Step-by-Step/schema/Product/) remains unchanged. It has no relationships of its own.

Additionally some relationships have been defined to linked to `https://schema.org/Person` entities.

### Designing Models for Linked Data.

Every entity relationship in NGSI-LD is a direct directional link from one entity to another.

Unlike the looser relationships in NGSI-v2, you should not place information

so for unlike

Can;t jump.

## Traversing links.

> **Example**: Imagine the scenario where a pallet of Products are moved from stock in the warehouse (`stockCount`) onto the shelves of
> the store (`storeCount`) . How would NGSI v2 and NGSI-LD computations differ?

### How is this defined in NGSI-v2?

In NGSI v2 the convenience bridge table **InventoryItem** entity had been created specifically to hold both count on the
shelf and count in the warehouse in a single location. In any computation only the **InventoryItem** entity would be
involved. The `stockCount` value would be decremented and the `shelfCount` value would incremented. In the NGSI v2 model
both the `storeCount` and the `shelfCount` have been placed into the conceptual **InventoryItem** Entity. This is a necessary workaround for NGSI v2 and it allows for simpler data reading and data
manipulation. However technically it is ontologically incorrect, as there is no such thing as an  **InventoryItem** in the real world, it is really two separate ledgers, products bought for the store and products sold on the shelf,
which in turn have an indirect relationship.

### How is this defined in NGSI-LD?

With linked data concepts (specifically `@graph` and `@context`) it is much easier for computers to understand indirect relationships and navigate between linked entities. Therefore
**Shelf** can be directly assigned a `numberOfItems` attribute and the model is ontologically correct.

Meanwhile another ontologically correct **StockOrder** Entity can be created
which holds a entry of which items are currently on order for each store. This is a proper context data entity as `stockCount` describes the current state of a product in the warehouse.

To move a pallet of products onto a shelf it would be possible for a computer navigate the relationships in the linked
data the `@graph` from **StockOrder** to **Shelf** as shown:

-   Some `product:XXX` items have been removed from `stockOrder:0001` - decrement `stockCount`.
-   Interogating the **StockOrder** is discovered that the **Product** is `requestedFor` for a specific URI e.g. `store:002`

```json
  "@graph": [
   {
      "@id": "tutorial:orderedProduct",
      "@type": "https://uri.etsi.org/ngsi-ld/Relationship",
      "schema:domainIncludes": [{"@id": "tutorial:StockOrder"}],
      "schema:rangeIncludes": [{"@id": "tutorial:Product"}],
      "rdfs:comment": "The Product ordered for a store",
      "rdfs:label": "orderedProduct"
    },
    ...etc
]
```
-   It is also discovered from the **StockOrder** model that the `requestedFor` URI defines a **Building**

```json
  "@graph": [
    {
      "@id": "tutorial:requestedFor",
      "@type": "https://uri.etsi.org/ngsi-ld/Relationship",
      "schema:domainIncludes": [{"@id": "tutorial:StockOrder"}],
      "schema:rangeIncludes": [{"@id": "fiware:Building"}],
      "rdfs:comment": "Store for which an item is requested",
      "rdfs:label": "requestedFor"
    },
    ...etc
]
```

-   It is discovered from the **Building** model that every **Building** contains `furniture` as an array of URIs.
-   It is discovered from the **Building** model that these URIs represent **Shelf** units

```json
"@graph": [
    {
      "@id": "tutorial:furniture",
      "@type": "https://uri.etsi.org/ngsi-ld/Relationship",
      "schema:domainIncludes": [{"@id": "fiware:Building"}],
      "schema:rangeIncludes": [{"@id": "tutorial:Shelf"}],
      "rdfs:comment": "Units found within a Building",
      "rdfs:label": "furniture"
    },
    ...etc
]
```


-   It is discovered from the **Shelf** model that the `stocks` attribute holds a URI representing **Product** items.

```json
"@graph": [
    {
      "@id": "tutorial:stocks",
      "@type": "https://uri.etsi.org/ngsi-ld/Relationship",
      "schema:domainIncludes": [{"@id": "tutorial:Shelf"}],
      "schema:rangeIncludes": [{"@id": "tutorial:Product"}],
      "rdfs:comment": "The product found on a shelf",
      "rdfs:label": "stocks"
    },
    ...etc
]
```


-   A request the **Shelf** unit which holds the correct **Product** for the `stocks` attribute is made and the Shelf `numberOfItems` attribute can be incremented.

The @grpah





# Prerequisites

## Docker

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Identity-Management/master/docker-compose.yml) is used
configure the required services for the application. This means all container services can be brought up in a single
command. Docker Compose is installed by default as part of Docker for Windows and Docker for Mac, however Linux users
will need to follow the instructions found [here](https://docs.docker.com/compose/install/)

## Cygwin

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/)
to provide a command-line functionality similar to a Linux distribution on Windows.

# Architecture

The demo application will send and receive NGSI-LD calls to a compliant context broker. Since both NSGI v2 and NSGI-LD
interfaces are available to an experimental version fo the
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/), our demo application will only make use of one
FIWARE component.

Currently, the Orion Context Broker relies on open source [MongoDB](https://www.mongodb.com/) technology to keep
persistence of the context data it holds. Therefore, the architecture will consist of two elements:

-   The [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json)
-   The underlying [MongoDB](https://www.mongodb.com/) database :
    -   Used by the Orion Context Broker to hold context data information such as data entities, subscriptions and
        registrations

Since all interactions between the two elements are initiated by HTTP requests, the elements can be containerized and
run from exposed ports.

![](https://fiware.github.io/tutorials.Relationships-Linked-Data/img/architecture.png)

The necessary configuration information can be seen in the services section of the associated `docker-compose.yml` file:

```yaml
orion:
    image: fiware/orion-ld
    hostname: orion
    container_name: fiware-orion
    depends_on:
        - mongo-db
    networks:
        - default
    ports:
        - "1026:1026"
    command: -dbhost mongo-db -logLevel DEBUG
    healthcheck:
        test: curl --fail -s http://orion:1026/version || exit 1
```

```yaml
mongo-db:
    image: mongo:3.6
    hostname: mongo-db
    container_name: db-mongo
    expose:
        - "27017"
    ports:
        - "27017:27017"
    networks:
        - default
    command: --nojournal
```

Both containers are residing on the same network - the Orion Context Broker is listening on Port `1026` and MongoDB is
listening on the default port `27071`. Both containers are also exposing the same ports externally - this is purely for
the tutorial access - so that cUrl or Postman can access them without being part of the same network. The command-line
initialization should be self explanatory.

The only notable difference to the introductory tutorials is that the required image name is currently
`fiware/orion-ld`.

# Start Up

All services can be initialised from the command-line by running the
[services](https://github.com/FIWARE/tutorials.Relationships-Linked-Data/blob/master/services) Bash script provided
within the repository. Please clone the repository and create the necessary images by running the commands as shown:

```bash
git clone git@github.com:FIWARE/tutorials.Relationships-Linked-Data.git
cd tutorials.Relationships-Linked-Data

./services start
```

> **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```
> ./services stop
> ```

---

---

## License

[MIT](LICENSE) © 2019 FIWARE Foundation e.V.
