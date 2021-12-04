

### 要解决的问题

--- 验证某人对某个实体有没有权限操作或者查看，比如5班的班主任可以查看该班级的学生的成绩，但

不能看别的班的学生的成绩。要解决谁对什么资源有什么访问权限的问题。



### spring acl https://www.baeldung.com/spring-security-acl

#### ACL_CLASS表

````
ACL_CLASS, which store class name of the domain object, columns include:

ID
CLASS: the class name of secured domain objects
````

#### ACL_SID表（表示权限表）

```
ACL_SID table which allows us to universally identify any principle or authority in the system. The table needs:

ID
SID: which is the username or role name. SID stands for Security Identity
PRINCIPAL: 0 or 1, to indicate that the corresponding SID is a principal (user, such as mary, mike, jack…) or an authority (role, such as ROLE_ADMIN, ROLE_USER, ROLE_EDITOR…)
```

#### ACL_OBJECT_IDENTITY表（记录SID与实体对象的关系）

```
ACL_OBJECT_IDENTITY, which stores information for each unique domain object:

ID
OBJECT_ID_CLASS: define the domain object class, links to ACL_CLASS table
OBJECT_ID_IDENTITY: domain objects can be stored in many tables depending on the class. Hence, this field store the target object primary key
PARENT_OBJECT: specify parent of this Object Identity within this table
OWNER_SID: ID of the object owner, links to ACL_SID table
ENTRIES_INHERITTING: whether ACL Entries of this object inherits from the parent object (ACL Entries are defined in ACL_ENTRY table)
```

#### ACL_ENTRY表

```
ACL_ENTRY store individual permission assigns to each SID on an Object Identity:

ID
ACL_OBJECT_IDENTITY: specify the object identity, links to ACL_OBJECT_IDENTITY table
ACE_ORDER: the order of current entry in the ACL entries list of corresponding Object Identity
SID: the target SID which the permission is granted to or denied from, links to ACL_SID table
MASK: the integer bit mask that represents the actual permission being granted or denied
GRANTING: value 1 means granting, value 0 means denying
AUDIT_SUCCESS and AUDIT_FAILURE: for auditing purpose
```

sid是否对object_id_identity有某种权限，这种权限存在ACL_ENTRY表中。