## JPA Entity Relationships
In a relational database, the relationships between two tables are defined by foreign keys. Typically, one table has a column that contains the primary key of another table’s row. In JPA, we deal with entity objects that are Java representations of database tables. So we need a different way for establishing relationship between two entities. JPA entity relationships define how these entities refer to each other.

For the purpose of this article, I will work with JPA 2.0 and a table structure as following.
```sql
CREATE TABLE team(team_id NUMBER, name VARCHAR2(20));
INSERT INTO team(team_id, name) VALUES(1, 'india');
INSERT INTO team(team_id, name) VALUES(2, 'australia');
INSERT INTO team(team_id, name) VALUES(3, 'england');


CREATE TABLE player(player_id NUMBER, name VARCHAR2(50), team_id NUMBER, age NUMBER, role VARCHAR2(20));
INSERT INTO player(player_id, name, team_id, age, role) VALUES(1, 'sachin', 1, 42, 'batsman');
INSERT INTO player(player_id, name, team_id, age, role) VALUES(2, 'dhoni', 1, 34, 'wicketkeeper');
INSERT INTO player(player_id, name, team_id, age, role) VALUES(3, 'clarke', 2, 38, 'batsman');
INSERT INTO player(player_id, name, team_id, age, role) VALUES(4, 'rogers', 2, 40, 'batsman');
INSERT INTO player(player_id, name, team_id, age, role) VALUES(5, 'cook', 3, 31, 'batsman');
INSERT INTO player(player_id, name, team_id, age, role) VALUES(6, 'root', 3, 26, 'batsman');

CREATE TABLE player_stat(player_stat_id NUMBER, player_id NUMBER, runs NUMBER, wickets NUMBER);
INSERT INTO player_stat(player_stat_id, player_id, runs, wickets) VALUES(10, 1, 10000, 300);
INSERT INTO player_stat(player_stat_id, player_id, runs, wickets) VALUES(20, 2, 5000, 10);
INSERT INTO player_stat(player_stat_id, player_id, runs, wickets) VALUES(30, 3, 7000, 100);
INSERT INTO player_stat(player_stat_id, player_id, runs, wickets) VALUES(40, 4, 2000, 0);
INSERT INTO player_stat(player_stat_id, player_id, runs, wickets) VALUES(50, 5, 9000, 0);
```
A team can have multiple players. A player can have a statistic.

### One to one
Each player has a player statistic, unless the player hasn’t made his debut yet. So we have a one to one relationship between a player and his statistics, and a `Player` entity will have a dependency on a `PlayerStat` instance. In relational database, this relationship could have been expressed by a foreign key in `player_stat` table, referring to a primary key in `player` table. But in object oriented paradigm, a `Player` object will have a reference of a `PlayerStat` instance. We can create our entities as following:
```java
@Entity
@Table(name="player")
public class Player {
  @Id
  @Column(columnDefinition="player_id", name="player_id")
  private Long id;
  
  private String name;
  private Long age;
  private String role;
  
  @OneToOne(mappedBy="player")
  @JsonManagedReference
  private PlayerStat playerStat;
  
  // getters and setters
}
```
```java
@Entity
@Table(name="player_stat")
public class PlayerStat {
  @Id
  @Column(columnDefinition="player_stat_id", name="player_stat_id")
  private Long id;
    
  private Long runs;
  private Long wickets;

  @OneToOne
  @JoinColumn(name="player_id")
  @JsonBackReference
  private Player player;
  
  // getters and setters
}
```
Notice that in our table structure, `player_stat` has a reference of `player` table, thus making it the *owner* of the relationship between these two tables. Both entities in the above code have a field of the opposite type with `@OneToOne` annotation. The owning side, i.e. `PlayerStat` entity declares the JOIN column. While the non-owning entity `Player` has a `mappedBy` attribute to establish the inverse relationship.

If both entities define a getter method for the member variable of the opposite type, we get a circular dependency if we try to serialize an object of either type. For this reason, we have used `@JsonManagedReference` and `@JsonBackReference` annotations. The former is used on a field that has to be serialized normally and the later is its counterpart that’s used on a field in the opposite type, that should be omitted from serialization.

### One to many/ Many to one
A team can have multiple players, while each player can belong to only one team, then we have a one to many relationship. In relational database, multiple `player` rows would have the reference of a `team` row via foreign key. But in Java, the relationship is represented by a `Team` object holding a reference to a list of `Player` objects belonging to that team.
```java
@Entity
@Table(name="team")
public class Team {
  @Id
  @Column(columnDefinition="team_id", name="team_id")
  private Long id;
  
  private String name;

  @OneToMany(mappedBy="team")
  @JsonManagedReference
  private List<Player> players;
  
  // getters and setters
}
```
```java
@Entity
@Table(name="player")
public class Player {
  @Id
  @Column(columnDefinition="player_id", name="player_id")
  private Long id;
  
  private String name;
  private Long age;
  private String role;
  
  @ManyToOne
  @JoinColumn(name="team_id")
  @JsonBackReference
  private Team team;
  
  // getters and setters
}
```
In this case, since `player` holds a reference of `team`, it is the owning side of their mutual relationship. `Team` declares its players field with `@OneToMany` annotation and conversely, `Player` has a reference of `Team` under `@ManyToOne` annotation. Since, `Player` is the owning side, it declares the JOIN column with `@JoinColumn` annotation and `Table` has the `mappedBy` attribute.

### Many to many
Suppose that the table `team` contains both national teams and private franchises. So that a team can have multiple players but an individual player can also belong to multiple teams. This relationship cannot be expressed with simple foreign keys between the two tables. Now we need a third table to hold the relationship. Our table structure will now look like:
```sql
CREATE TABLE team(team_id NUMBER, name VARCHAR2(20));
INSERT INTO team(team_id, name) VALUES(1, 'India');
INSERT INTO team(team_id, name) VALUES(2, 'Australia');
INSERT INTO team(team_id, name) VALUES(3, 'Sydney');
INSERT INTO team(team_id, name) VALUES(4, 'Pune');


CREATE TABLE player(player_id NUMBER, name VARCHAR2(50), age NUMBER, role VARCHAR2(20));
INSERT INTO player(player_id, name, age, role) VALUES(10, 'sachin', 42, 'batsman');
INSERT INTO player(player_id, name, age, role) VALUES(20, 'yuvraj', 37, 'allrounder');
INSERT INTO player(player_id, name, age, role) VALUES(30, 'smith', 28, 'batsman');

CREATE TABLE team_player(team_id NUMBER, player_id NUMBER);
INSERT INTO team_player(team_id, player_id) VALUES(1, 10);
INSERT INTO team_player(team_id, player_id) VALUES(1, 20);
INSERT INTO team_player(team_id, player_id) VALUES(2, 30);
INSERT INTO team_player(team_id, player_id) VALUES(3, 30);
INSERT INTO team_player(team_id, player_id) VALUES(4, 20);
INSERT INTO team_player(team_id, player_id) VALUES(4, 30);
```
In many to many, both entities will have a reference to a list of the other type.
```java
@Entity
@Table(name="team")
public class Team {
  @Id
  @Column(columnDefinition="team_id", name="team_id")
  private Long id;
  
  private String name;

  @ManyToMany
  @JoinTable(name="team_player", joinColumns={@JoinColumn(name="team_id")}, inverseJoinColumns={@JoinColumn(name="player_id")})
  private List<Player> players;
  
  // getters and setters
}
```
```java
@Entity
@Table(name="player")
public class Player {
  @Id
  @Column(columnDefinition="player_id", name="player_id")
  private Long id;
  
  private String name;  
  private Long age;
  private String role;
  
  @ManyToMany(mappedBy="players")
  private List<Team> teams;
  
  // getters and setters
}
```
Both entities use `@ManyToMany` annotation. Since relationship is stored in a third table, neither team or player has the ownership. We can declare the join table in either entity. In `@JoinTable` annotation, we specify `joinColumns` attribute to declare the relationship between the join table and the table corresponding to this entity. `inverseJoinColumns` is used to declare the relationship between the join table and the other table. We declared `@JoinTable` in `Team` entity so `Player` has `mappedBy` attribute.

### Embeddable
If our entity has lots of attributes, we may group some of those attributes and separate them out in separate logical entities.

In a typical example, suppose we have an `employee` table which among other things, also holds an employee’s address with help of columns like `house_number`, `street`, `locality` etc. Having all these attributes in the `Employee` entity may clutter up the entity class. In such a case we can move all address specific attributes to a separate `Address` entity. Our `Employee` will now hold just a single reference to an `Address` entity. Note that the `Address` entity is not backed by any database table, neither does it has any id attribute. So an `Address` entity object can be queried or persisted only in the context of its parent `Employee` instance.

Now lets apply this concept to our player model. In `player` table we have `age` and `role` columns and we have corresponding attributes in the `Player` entity. We can move these two columns to a new `Profile` entity as declared below.
```java
@Embeddable
public class Profile {
  private Long age;
  private String role;

  // getters and setters
}
```
And we can modify our `Player` class as following:
```java
@Entity
@Table(name="player")
public class Player {
  @Id
  @Column(columnDefinition="player_id", name="player_id")
  private Long id;
  
  private String name;
  
  private Profile profile;
  
  // getters and setters
}
```
This way our entities become more focused.

