一共以下5个表

- TTP_USER：用户表。
- TTP_BOOK：图书详情表。记录图书信息和库存。
- TTP_BOOK_HISTORY：借阅历史表。用户预约借，管理员确认借，管理员还书
- TTP_BOOK_STAR：踩赞表。统计踩赞信息，用户图书排行榜
- TTP_BOOK_STAR_HISTORY：踩赞记录表。记录踩赞历史，避免重复操作

# TTP_USER：用户表

| 字段        | 字段类型   | 说明                              |
| ----------- | ---------- | --------------------------------- |
| ID          | Long       | 用户ID，同：userId                |
| UNAME       | String     | 姓名，用于显示，不用于登陆        |
| EMAIL       | String     | 邮箱，用于登陆，同：参数 username |
| PASSWD      | String     | 密码，MD5存储，同：参数 password  |
| CREATE_TIME | Date、Long | 创建时间                          |
| MODIFY_TIME | Date、Long | 更新时间                          |

# TTP_BOOK：图书详情表

| 字段        | 字段类型   | 说明                                                         |
| ----------- | ---------- | ------------------------------------------------------------ |
| ID          | Long       | 图书ID，bookId                                               |
| ISBN13      | String     | 13位 ISBN                                                    |
| ISBN10      | String     | 10位 ISBN                                                    |
| TITLE       | String     | 书名                                                         |
| AUTHOR      | String     | 作者                                                         |
| PUBLISHER   | String     | 出版社                                                       |
| SUMMARY     | String     | 图书概要                                                     |
| ALT         | String     | 豆瓣链接                                                     |
| PRICE       | String     | 价格                                                         |
| IMAGE       | String     | 照片                                                         |
| STOCK       | Integer    | 原始库存，该数字不变                                         |
| LEND        | Integer    | 借出了多少。实际上还有个虚拟字段 left = STOCK-LEND，剩余库存 |
| CREATE_TIME | Date、Long | 创建时间                                                     |
| MODIFY_TIME | Date、Long | 更新时间                                                     |

# TTP_BOOK_HISTORY：借阅历史表

| 字段        | 字段类型   | 说明                                                         |
| ----------- | ---------- | ------------------------------------------------------------ |
| ID          | Long       | historyId 借阅历史ID                                         |
| BOOK_ID     | Long       | bookId 图书ID                                                |
| USER_ID     | Long       | userId 用户ID                                                |
| STATE       | Integer    | 借阅状态 (0:已借待确认，10:确认已借，20:确认已还)<br />stateText，用于前端显示 |
| CREATE_TIME | Date、Long | 创建时间、借阅时间                                           |
| MODIFY_TIME | Date、Long | 更新时间、还书时间(状态是20的时候才显示)                     |

# TTP_BOOK_STAR：踩赞表

| 字段        | 字段类型   | 说明                                        |
| ----------- | ---------- | ------------------------------------------- |
| ID          | Long       |                                             |
| BOOK_ID     | Long       | booId图书ID                                 |
| UP          | Integer    | 赞                                          |
| DOWN        | Integer    | 踩                                          |
| STAR        | Integer    | 综合评级 STAR = UP - DOWN，出现负数是正常的 |
| CREATE_TIME | Date、Long | 创建时间                                    |
| MODIFY_TIME | Date、Long | 更新时间                                    |

# TTP_BOOK_STAR_HISTORY：踩赞记录表

| 字段        | 字段类型   | 说明                                                 |
| ----------- | ---------- | ---------------------------------------------------- |
| ID          | Long       | 无用                                                 |
| USER_ID     | Long       | 用户ID                                               |
| BOOK_ID     | Long       | 图书ID                                               |
| ACTION      | Integer    | 踩赞行为（0:up，1：down）                            |
| DELETED     | Integer    | 取消踩，或者取消赞，记录标记为删除（1:删除，0:默认） |
| CREATE_TIME | Date、Long | 创建时间                                             |
| MODIFY_TIME | Date、Long | 更新时间                                             |

