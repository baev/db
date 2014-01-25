## route_db, база маршрутов городского транспорта.

### Таблицы

Маршрут. **id**, и **type** (*bus, tram etc*).

```sql
CREATE TABLE IF NOT EXISTS `route_db`.`route` (
  `id` INT UNSIGNED NOT NULL,
  `type` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE INDEX `id_UNIQUE` (`id` ASC))
ENGINE = InnoDB;
```

Остановки. **id**, **name** - название остановки, **address** - адрес остановки, **type** - тип остановки (*bus stop, tram stop etc*)

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

Остановки маршрута. **stop_number** - порядковый номер остановки в маршруте, **route_id** - индификатор маршрута, **stop_id** индификатор остановки.

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

