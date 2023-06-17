# BrewHub.Net: Distillery Edge Configuration

This section defines a docker composition which runs on the edge device, communicating
with leaf devices on the same network.

Currently, there is no solution-specific code running on the edge layer. But if there is
at some point, this is where it would reside.

## Edge stack

To bring up the edge stack...

```
docker compose up
```

## Simulated controllers

To add simulated controllers to the mix, also include the `docker-compose-controllers.yml` file.

```
docker compose -f docker-compose.yml -f docker-compose-controllers.yml up
```