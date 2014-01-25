## route_db, база маршрутов городского транспорта.

### Tables

Маршрут. **id**, и **type** (*bus, tram etc*).

```sql
CREATE TABLE IF NOT EXISTS `route_db`.`route` (
  `id` INT UNSIGNED NOT NULL,
  `type` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE INDEX `id_UNIQUE` (`id` ASC))
ENGINE = InnoDB;
```

Остановки. **id**, **name** - название остановки, **address** - адрес остановки, **type** - тип остановки (*bus stop, tram stop etc*).

```sql
CREATE TABLE IF NOT EXISTS `route_db`.`stop` (
  `id` INT UNSIGNED NOT NULL,
  `name` VARCHAR(45) NOT NULL,
  `address` VARCHAR(45) NOT NULL,
  `type` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE INDEX `id_UNIQUE` (`id` ASC))
ENGINE = InnoDB;
```

Остановки маршрута. **stop_number** - порядковый номер остановки в маршруте, **route_id** - идентификатор маршрута, **stop_id** идентификатор остановки.

```sql
CREATE TABLE IF NOT EXISTS `route_db`.`route_stop` (
  `stop_number` INT UNSIGNED NOT NULL,
  `route_id` INT UNSIGNED NOT NULL,
  `stop_id` INT UNSIGNED NOT NULL,
  INDEX `fk_route_stop_stop1_idx` (`stop_id` ASC),
  CONSTRAINT `fk_route_stop_route1`
    FOREIGN KEY (`route_id`)
    REFERENCES `route_db`.`route` (`id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_route_stop_stop1`
    FOREIGN KEY (`stop_id`)
    REFERENCES `route_db`.`stop` (`id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;
```

Транспорт. **id**, **type** - тип транспорта (*bus, tram etc*), **route_id** - идентификатор маршрута, **vehicle_park_id** идентификатор депо.
 
```sql
CREATE TABLE IF NOT EXISTS `route_db`.`vehicle` (
  `id` INT UNSIGNED NOT NULL,
  `type` VARCHAR(45) NOT NULL,
  `route_id` INT UNSIGNED NOT NULL,
  `vehicle_park_id` INT NOT NULL,
  PRIMARY KEY (`id`),
  INDEX `fk_vehicle_route_idx` (`route_id` ASC),
  INDEX `fk_vehicle_vehicle_park1_idx` (`vehicle_park_id` ASC),
  CONSTRAINT `fk_vehicle_route`
    FOREIGN KEY (`route_id`)
    REFERENCES `route_db`.`route` (`id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_vehicle_vehicle_park1`
    FOREIGN KEY (`vehicle_park_id`)
    REFERENCES `route_db`.`vehicle_park` (`id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;
```

Депо. **id**, **name** - название остановки, **address** - адрес остановки, **type** - тип остановки (*bus stop, tram stop etc*).

```sql
CREATE TABLE IF NOT EXISTS `route_db`.`vehicle_park` (
  `id` INT NOT NULL,
  `name` VARCHAR(45) NOT NULL,
  `address` VARCHAR(45) NOT NULL,
  `type` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`id`))
ENGINE = InnoDB;
```

Водители. **id**, **name** - имя водителя, **surname** - фамилия водителя.

```sql
CREATE TABLE IF NOT EXISTS `route_db`.`driver` (
  `id` INT NOT NULL,
  `name` VARCHAR(45) NOT NULL,
  `surname` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`id`))
ENGINE = InnoDB;
```

Водительские смены. **date** - дата смены, **vehicle_id** - идентификатор транспортного средства, **driver_id** идентификатор водителя.

```sql
CREATE TABLE IF NOT EXISTS `route_db`.`driver_shift` (
  `date` DATETIME NOT NULL,
  `vehicle_id` INT UNSIGNED NOT NULL,
  `driver_id` INT NOT NULL,
  INDEX `fk_driver_shift_driver1_idx` (`driver_id` ASC),
  CONSTRAINT `fk_driver_shift_vehicle1`
    FOREIGN KEY (`vehicle_id`)
    REFERENCES `route_db`.`vehicle` (`id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_driver_shift_driver1`
    FOREIGN KEY (`driver_id`)
    REFERENCES `route_db`.`driver` (`id`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;
```

### Views

Маршруты. **route_id** - номер маршрута, **stop_number** - порядковый номер остановки на маршруте. **stop_id** - идентификатор остановки, **name** - название остановки.

```sql
CREATE VIEW `route_list` AS
SELECT route.id, stop_number, name, route.type FROM route, route_stop, stop WHERE route_id=route.id AND stop_id=stop.id ORDER BY route.id, stop_number;
```
