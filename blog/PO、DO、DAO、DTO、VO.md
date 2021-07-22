## PO

Persistant Object，持久对象，对应数据库中的表结构

## DO

Domain Object，领域对象，对应数据库中的表结构。分不清和PO的区别，我觉得有点像是一个东西

## DAO

Data Access Object，数据访问对象，负责持久层的操作。通过它配合PO对数据库进行增删查改

## DTO

Data Transfer Object，数据传输对象，通常用于不同服务之间或者同一服务的不同分层之间的数据传输，比PO的属性要少。

## VO

View Object，视图对象，把相关数据传输给前端显示。比如PO中的密码不能传输，那么VO中就不会封装密码，只封装前端需要的属性。比PO的属性要少。

## BO

Business Object，业务对象，处于业务层，包含多个类。比如一个用户关联了个人信息和家庭信息等类