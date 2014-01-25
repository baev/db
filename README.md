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

Маршруты. **id** - номер маршрута, **stop_number** - порядковый номер остановки на маршруте. **stop_id** - идентификатор остановки, **name** - название остановки.

```sql
CREATE VIEW `route_list` AS
    SELECT 
        route.id, stop_number, name, route.type
    FROM
        route,
        route_stop,
        stop
    WHERE
        route_id = route.id
            AND stop_id = stop.id
    ORDER BY route.id , stop_number;
```

### Triggers

Проверяет добавляемые в маршрут остановки. 
1. Тип остановки должен совпадать с типом маршрута
2. Добавляемая в маршрут остановка должна иметь следующий порядковый номер, или быть 1й.

```sql
USE `route_db`;
DROP trigger IF EXISTS `route_db`.`route_stop_trigger`;

DELIMITER $$
CREATE TRIGGER `route_stop_trigger` BEFORE INSERT ON route_stop
for each row
BEGIN

        if (select type from route where route.id=new.route_id) <> (select type from stop where stop.id=new.stop_id) then
                set @msg = "Wrong stop type on route";
                SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = @msg;
        end if;

        if ((select max(stop_number) from route_stop where route_stop.route_id=new.route_id)<>new.stop_number-1) then
                set @msg = "Wrong stop_number";
                SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = @msg;
        end if;

        if ((select count(stop_number) from route_stop where route_stop.route_id=new.route_id)=0 and new.stop_number<>1) then
                set @msg = "Wrong stop_number";
                SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = @msg;
        end if;
END$$

DELIMITER ;

```

### Test data

Создаем городские маршруты.

```sql
START TRANSACTION;
USE `route_db`;
INSERT INTO `route_db`.`route` (`id`, `type`) VALUES (1, 'bus');
INSERT INTO `route_db`.`route` (`id`, `type`) VALUES (2, 'bus');
INSERT INTO `route_db`.`route` (`id`, `type`) VALUES (3, 'tram');
INSERT INTO `route_db`.`route` (`id`, `type`) VALUES (4, 'tram');

COMMIT;
```

Создаем остановки.

```sql
START TRANSACTION;
USE `route_db`;
INSERT INTO `route_db`.`stop` (`id`, `name`, `address`, `type`) VALUES (1, 'ул. Попова', 'spb', 'bus');
INSERT INTO `route_db`.`stop` (`id`, `name`, `address`, `type`) VALUES (2, 'Библиотека', 'spb', 'bus');
INSERT INTO `route_db`.`stop` (`id`, `name`, `address`, `type`) VALUES (3, 'Площадь Восстания', 'spb', 'bus');
INSERT INTO `route_db`.`stop` (`id`, `name`, `address`, `type`) VALUES (4, 'ул. ак. Павлова', 'spb', 'bus');
INSERT INTO `route_db`.`stop` (`id`, `name`, `address`, `type`) VALUES (5, 'Университет', 'spb', 'tram');
INSERT INTO `route_db`.`stop` (`id`, `name`, `address`, `type`) VALUES (6, 'Кладбище', 'spb', 'tram');
INSERT INTO `route_db`.`stop` (`id`, `name`, `address`, `type`) VALUES (7, 'Морг', 'spb', 'tram');
INSERT INTO `route_db`.`stop` (`id`, `name`, `address`, `type`) VALUES (8, 'Школа', 'spb', 'bus');

COMMIT;
```

Наполняем маршруты остановками.

```sql
START TRANSACTION;
USE `route_db`;
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (1, 1, 4);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (2, 1, 2);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (3, 1, 1);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (4, 1, 8);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (5, 1, 3);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (1, 2, 4);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (2, 2, 1);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (3, 2, 8);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (4, 2, 4);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (1, 3, 5);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (2, 3, 6);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (3, 3, 7);
INSERT INTO `route_db`.`route_stop` (`stop_number`, `route_id`, `stop_id`) VALUES (4, 3, 5);

COMMIT;
```

Создаем два депо, для трамваев и автобусов.

```sql
START TRANSACTION;
USE `route_db`;
INSERT INTO `route_db`.`vehicle_park` (`id`, `name`, `address`, `type`) VALUES (1, 'южный', 'spb', 'bus');
INSERT INTO `route_db`.`vehicle_park` (`id`, `name`, `address`, `type`) VALUES (2, 'северный', 'spb', 'tram');

COMMIT;
```

Создаем автопарк.

```sql
START TRANSACTION;
USE `route_db`;
INSERT INTO `route_db`.`vehicle` (`id`, `type`, `route_id`, `vehicle_park_id`) VALUES (1, 'bus', 1, 1);
INSERT INTO `route_db`.`vehicle` (`id`, `type`, `route_id`, `vehicle_park_id`) VALUES (2, 'bus', 2, 1);
INSERT INTO `route_db`.`vehicle` (`id`, `type`, `route_id`, `vehicle_park_id`) VALUES (3, 'bus', 1, 1);
INSERT INTO `route_db`.`vehicle` (`id`, `type`, `route_id`, `vehicle_park_id`) VALUES (4, 'bus', 2, 1);
INSERT INTO `route_db`.`vehicle` (`id`, `type`, `route_id`, `vehicle_park_id`) VALUES (5, 'tram', 3, 2);
INSERT INTO `route_db`.`vehicle` (`id`, `type`, `route_id`, `vehicle_park_id`) VALUES (6, 'tram', 3, 2);

COMMIT;
```

Создаем водителей.

```sql
START TRANSACTION;
USE `route_db`;
INSERT INTO `route_db`.`driver` (`id`, `name`, `surname`) VALUES (1, 'Юля', 'Красивая');
INSERT INTO `route_db`.`driver` (`id`, `name`, `surname`) VALUES (2, 'Маша', 'Светлая');
INSERT INTO `route_db`.`driver` (`id`, `name`, `surname`) VALUES (3, 'Света', 'Добрая');
INSERT INTO `route_db`.`driver` (`id`, `name`, `surname`) VALUES (4, 'Даша', 'Милая');
INSERT INTO `route_db`.`driver` (`id`, `name`, `surname`) VALUES (5, 'Аня', 'Умная');
INSERT INTO `route_db`.`driver` (`id`, `name`, `surname`) VALUES (6, 'Лиза', 'Гибкая');
INSERT INTO `route_db`.`driver` (`id`, `name`, `surname`) VALUES (7, 'Валя', 'Страшная');

COMMIT;
```

Распределяем смены водителям.

```sql
START TRANSACTION;
USE `route_db`;
INSERT INTO `route_db`.`driver_shift` (`date`, `vehicle_id`, `driver_id`) VALUES ('2014-01-02', 1, 1);
INSERT INTO `route_db`.`driver_shift` (`date`, `vehicle_id`, `driver_id`) VALUES ('2014-01-02', 2, 2);
INSERT INTO `route_db`.`driver_shift` (`date`, `vehicle_id`, `driver_id`) VALUES ('2014-01-02', 3, 3);
INSERT INTO `route_db`.`driver_shift` (`date`, `vehicle_id`, `driver_id`) VALUES ('2014-01-02', 4, 4);
INSERT INTO `route_db`.`driver_shift` (`date`, `vehicle_id`, `driver_id`) VALUES ('2014-01-02', 5, 5);
INSERT INTO `route_db`.`driver_shift` (`date`, `vehicle_id`, `driver_id`) VALUES ('2014-01-02', 6, 6);

COMMIT;
```



