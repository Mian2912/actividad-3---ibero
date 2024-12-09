# Actividad 3 - conceptos y comandos basicos del particionamiento en base de datos nosql

Comandos para el desarrollo de la actividad 3 base de datos sobre el particionamiento sobre mongodb

- Ejecuta este comando en la terminal para iniciar el Config Server en el puerto 27019
    ```bash
    mongod --configsvr --replSet configReplSet --port 27019 --dbpath ~/partitions/configdb --bind_ip localhost
    ```
- conectarse al config server en el puerto 27019
    ```bash
    mongosh --port 27019
    ```
- Ejecuta el comando:
    ```bash
    rs.initiate({
        _id: "configReplSet",
        configsvr: true,
        members: [{ _id: 0, host: "localhost:27019" }]
    });
    ```
## configuracion de los shards
- configuracion para el shard 1
    ```bash
    mongod --shardsvr --replSet shard1ReplSet --port 27018 --dbpath ~/partitions/shard1 --bind_ip localhost
    ```
- configuracion para el shard 2
    ```bash
    mongod --shardsvr --replSet shard2ReplSet --port 27017 --dbpath ~/partitions/shard2 --bind_ip localhost
    ```
### Inicializar las replica sets de los shards
- conectarse al shard 1
    ```bash
    mongosh --port 27018
    ```
- ejecutar el siguiente comando
    ```bash
    rs.initiate({
        _id: "shard1ReplSet",
        members: [{ _id: 0, host: "localhost:27018" }]
    });
    ```
- conectarse al shard 2
    ```bash
    mongosh --port 27017
    ```
- ejecutar el siguiente comando
    ```bash
    rs.initiate({
        _id: "shard2ReplSet",
        members: [{ _id: 0, host: "localhost:27017" }]
    });
    ```

## configuracion del router
- Ejecuta este comando en la terminal para iniciar el router en el puerto 27020
    ```bash
    mongod --configdb configReplSet/localhost:27019 --port 27020 --bind_ip localhost
    ```
- conexion al puerto 27020
    ```bash
    mongosh --port 27020
    ```
- ejecutar los siguientes comandos
    ```bash
    sh.addShard("shard1ReplSet/localhost:27018");
    sh.addShard("shard2ReplSet/localhost:27017");
    ```
## activacion del sharding
- habilita el sharding para la base de datos
    ```bash
    sh.enableSharding("sports_event");
    ```
- Configura el particionamiento en la colección participants usando la clave event_id
    ```bash
    sh.shardCollection("sports_event.participants", { event_id: "hashed" });
    ```
## Verificacion del sharding
- revisar el estado del cluster
    ```bash
    sh.status();
    ```
- insercion de datos de prueba
    ```bash
    db.participants.insertMany([
        { event_id: 1, participant_id: 101, name: "Alice" },
        { event_id: 2, participant_id: 102, name: "Bob" },
        { event_id: 3, participant_id: 103, name: "Charlie" }
    ]);
    ```
- verificar la distribucion de datos
    ```bash
    db.participants.find().explain("executionStats");
    ```

