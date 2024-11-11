> 本章节以`MySQL8.x`为例，服务端语言采用`node`，调用`MySQL`的`npm`包为 `mysql2`

## 安装和配置

## 基础操作
### 查询
#### 不含条件查询表里的所有数据
```sql
语法： select * from [表名]

select * from user_list

```
#### 根据某一字段的某一个值进行查询(name为例)
```sql
语法： select * from [表名] where name = [值]

select * from user_list where name = 'web'

```
#### 根据某一个字段的某一个值进行模糊匹配

#### 根据多个字段的多个值进行匹配

> 表中有`name`和`age`字段，匹配一个数组中[{name: '', age}, {name: '', age: ''}]所有满足条件的值

```sql
select * from ad_rule_agent where (name, age) in ((1045,6568),(1045,6313))

```

node写法
const [list] = await pool_user.query('select * from ad_rule_agent where (name, age) in (?)', [agent_list.map(item => [item.channel_id, item.agent_id])])

### 增加

# 添加多条记录
insert into ad_rule_agent(channel_id, agent_id, type, proportion) values (1, 2, 3, 4), (1, 3, 3, 4)

await pool_dsp.query('insert into ad_rule_agent(channel_id, agent_id, type, proportion) values ?', [agent_list.map(item => [item.channel_id, item.agent_id, item.type, item.proportion])])


### 删除

### 修改

## 高级操作
