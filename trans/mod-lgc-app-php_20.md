# 附录 C. 网关后的代码

本附录显示了附录 B 中页面脚本在转换为使用网关类之后的版本。请注意，它几乎没有改变。尽管 SQL 语句已被移除，但领域业务逻辑仍嵌入在页面脚本中。

网关类在页面脚本下面提供，并显示了转换为 PDO 风格绑定参数。还要注意，页面脚本中的`if()`条件已经进行了微小的修改：以前它们检查查询是否成功，现在它们检查网关的返回值。

```php
**page_script.php**
<?php
2 // ... $user_id value created earlier
3
4 $db = new Database($db_host, $db_user, $db_pass);
5 $articles_gateway = new ArticlesGateway($db);
6 $users_gateway = new UsersGateway($db);
7
8 $article_types = array(1, 2, 3, 4, 5);
9 $failure = array();
10 $now = time();
11
12 // sanitize and escape the user input
13 $input = $_POST;
14 $input['body'] = strip_tags($input['body']);
15 $input['notes'] = strip_tags($input['notes']);
16
17 if (isset($input['ready']) && $input['ready'] == 'on') {
18 $input['ready'] = 1;
19 } else {
20 $input['ready'] = 0;
21 }
22
23 // nothing less than 0.01 credits per rating
24 $input['credits_per_rating'] = round(
25 $input['credits_per_rating'],
26 2
27 );
28
29 $credits = round(
30 $input['credits_per_rating'] * $input['max_ratings'],
31 2
32 );
33
34 // updating an existing article?
35 if ($input['id']) {
36
37 $row = $articles_gateway->selectOneByIdAndUserId($input['id'], $user_id);
38
39 if ($row) {
40
41 // don't charge unless the article is ready
42 $decrement = false;
43
44 // is the article marked as ready?
45 if ($input['ready'] == 1) {
46
47 // did they offer at least the minimum?
48 if (
49 $credits > 0
50 && $input['credits_per_rating'] >= 0.01
51 && is_numeric($credits)
52 ) {
53
54 // was the article previously ready for review?
55 // (note 'row' not 'input')
56 if ($row['ready'] == 1) {
57
58 // only subtract (or add back) the difference to their
59 // account, since they already paid something
60 if (
61 is_numeric($row['credits_per_rating'])
62 && is_numeric($row['max_ratings'])
63 ) {
64 // user owes $credits, minus whatever they paid already
65 $amount = $row['credits_per_rating']
66 * $row['max_ratings']
67 $credits = $credits - $amount;
68 }
69
70 $decrement = true;
71
72 } else {
73 // article not ready previously, so they hadn't
74 // had credits deducted. if this is less than their
75 // in their account now, they may proceed.
76 $residual = $user->get('credits') - $credits;
77 $decrement = true;
78 }
79
80 } else {
81 $residual = -1;
82 $failure[] = "Credit offering invalid.";
83 $decrement = false;
84 }
85
86 } else {
87
88 // arbitrary positive value; they can proceed
89 $residual = 1;
90
91 // if it was previously ready but is no longer, refund them
92 if (
93 is_numeric($row['credits_per_rating'])
94 && is_numeric($row['max_ratings'])
95 && ($row['ready'] == 1)
96 ) {
97 // subtract a negative value
98 $amount = $row['credits_per_rating']
99 * $row['max_ratings']
100 $credits = -($amount);
101 $decrement = true;
102 }
103 }
104
105 if ($residual >= 0) {
106
107 $input['ip'] = $_SERVER['REMOTE_ADDR'];
108 $input['last_edited'] = $now;
109
110 if (! in_array(
111 $input['article_type'],
112 $article_types
113 )) {
114 $input['article_type'] = 1;
115 }
116
117 $result = $articles_gateway->updateByIdAndUserId(
118 $input['id'],
119 $user_id,
120 $input
121 );
122
123 if ($result) {
124 $article_id = $input['id'];
125
126 if ($decrement) {
127 $users_gateway->decrementCredits($user_id, $credits);
128 }
129 } else {
130 $failure[] = "Could not update article.";
131 }
132 } else {
133 $failure[] = "You do not have enough credits for ratings.";
134 }
135 }
136
137 } else {
138
139 // creating a new article. do not decrement until specified.
140 $decrement = false;
141
142 // if the article is ready, we need to subtract credits.
143 if ($input['ready'] == 1) {
144
145 // if this is greater than or equal to 0, they may proceed.
146 if (
147 $credits > 0
148 && $input['credits_per_rating']>=0.01
149 && is_numeric($credits)
150 ) {
151 // minimum offering is 0.01
152 $residual = $user->get('credits') - $credits;
153 $decrement = true;
154 } else {
155 $residual = -1;
156 $failure[] = "Credit offering invalid.";
157 }
158
159 } else {
160 // arbitrary positive value if they are not done with their article.
161 // no deduction made yet.
162 $residual = 1;
163 }
164
165 // can user afford ratings on the new article?
166 if ($residual >= 0) {
167
168 // yes, insert the article
169 $input['last_edited'] = $now;
170 $input['ip'] = $_SERVER['REMOTE_ADDR'];
171 $article_id = $articles_gateway->insert($input);
172
173 if ($article_id) {
174 if ($decrement) {
175 // Charge them
176 $users_gateway->decrementCredits($user_id, $credits);
177 }
178 } else {
179 $failure[] = "Could not update credits.";
180 }
181
182 $result = $articles_gateway->updateByIdAndUserId(
183 $article_id,
184 $user_id,
185 $input
186 );
187
188 if (! $result) {
189 $failure[] = "Could not update article.";
190 }
191
192 } else {
193
194 // cannot afford ratings on new article
195 $failure[] = "You do not have enough credits for ratings.";
196 }
197 }
198 ?>
```

```php
**classes/Domain/Articles/ArticlesGateway.php**
1 <?php
2 namespace Domain\Articles;
3
4 class ArticlesGateway
5 {
6 protected $db;
7
8 public function __construct(Database $db)
9 {
10 $this->db = $db;
11 }
12
13 public function selectOneByIdAndUserId($id, $user_id)
14 {
15 $stm = "SELECT *
16 FROM articles
17 WHERE user_id = :user_id
18 AND id = :id
19 LIMIT 1";
20
21 return $this->db->query($stm, array(
22 'id' => $id,
23 'user_id' => $user_id,
24 ))
25 }
26
27 public function updateByIdAndUserId($id, $user_id, $input)
28 {
29 if (strlen($input['notes']) > 0) {
30 $notes = "notes = :notes";
31 } else {
32 $notes = "notes = NULL";
33 }
34
35 if (strlen($input['title']) > 0) {
36 $title = "title = :title";
37 } else {
38 $title = "title = NULL";
39 }
40
41 $input['id'] = $id;
42 $input['user_id'] = $user_id;
43
44 $stm = "UPDATE articles
45 SET
46 body = :body,
47 $notes,
48 $title,
49 article_type = :article_type,
50 ready = :ready,
51 last_edited = :last_edited,
52 ip = :ip,
53 credits_per_rating = :credits_per_rating,
54 max_ratings = :max_ratings
55 WHERE user_id = :user_id
56 AND id = :id";
57
58 return $this->query($stm, $input);
59 }
60
61 public function insert($input)
62 {
63 $stm = "INSERT INTO articles (
64 user_id,
65 ip,
66 last_edited,
67 article_type
68 ) VALUES (
69 :user_id,
70 :ip,
71 :last_edited,
72 :article_type
73 )";
74 $this->db->query($stm, $input);
75 return $this->db->lastInsertId();
76 }
77 }
78 ?>
```

```php
**classes/Domain/Users/UsersGateway.php**
1 <?php
2 namespace Domain\Users;
3
4 class UsersGateway
5 {
6 protected $db;
7
8 public function __construct(Database $db)
9 {
10 $this->db = $db;
11 }
12
13 public function decrementCredits($user_id, $credits)
14 {
15 $stm = "UPDATE users
16 SET credits = credits - :credits
17 WHERE user_id = :user_id";
18 $this->db->query($stm, array(
19 'user_id' => $user_id,
20 'credits' => $credits,
21 ));
22 }
23 }
24 ?>
```
