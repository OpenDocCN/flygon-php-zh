# 附录 D. 事务脚本后的代码

本附录展示了从附录 B 和 C 中提取领域逻辑到*Transactions*类的代码版本。请注意原始页面脚本现在被简化为对象创建和注入机制，并将大部分逻辑交给*Transactions*类处理。还请注意，现在`$failure`、`$credits`和`$article_types`变量现在是*Transactions*类的属性，而规范化/清理逻辑和信用计算逻辑是*Transactions*逻辑的一部分。

```php
**page_script.php**
<?php
2
3 // ... $user_id value created earlier
4
5 $db = new Database($db_host, $db_user, $db_pass);
6 $articles_gateway = new ArticlesGateway($db);
7 $users_gateway = new UsersGateway($db);
8 $article_transactions = new ArticleTransactions(
9 $articles_gateway,
10 $users_gateway
11 );
12
13 if ($_POST['id']) {
14 $article_transactions->updateExistingArticle($user_id, $_POST);
15 } else {
16 $article_transactions->submitNewArticle($user_id, $_POST);
17 }
18
19 $failure = $article_transactions->getFailure();
20 ?>
```

```php
**classes/Domain/Articles/ArticleTransactions.php**
1 <?php
2 namespace Domain\Articles;
3
4 use Domain\Users\UsersGateway;
5
6 class ArticleTransactions
7 {
8 protected $article_types = array(1, 2, 3, 4, 5);
9
10 protected $failure = array();
11
12 protected $input = array();
13
14 public function __construct(
15 ArticlesGateway $articles_gateway,
16 UsersGateway $users_gateway
17 ) {
18 $this->articles_gateway = $articles_gateway;
19 $this->users_gateway = $users_gateway;
20 }
21
22 public function getInput()
23 {
24 return $this->input;
25 }
26
27 public function getFailure()
28 {
29 return $this->failure;
30 }
31
32 public function getCredits()
33 {
34 return round(
35 $this->input['credits_per_rating'] * $this->input['max_ratings'],
36 2
37 );
38 }
39
40 public function filterInput($input)
41 {
42 $input['body'] = strip_tags($input['body']);
43 $input['notes'] = strip_tags($input['notes']);
44
45 if (isset($input['ready']) && $input['ready'] == 'on') {
46 $input['ready'] = 1;
47 } else {
48 $input['ready'] = 0;
49 }
50
51 // nothing less than 0.01 credits per rating
52 $input['credits_per_rating'] = round(
53 $input['credits_per_rating'],
54 2
55 );
56
57 // return the filtered input
58 return $input;
59 }
60
61 public function updateExistingArticle($user_id, $input)
62 {
63 $this->input = $this->filterInput($input);
64 $now = time();
65 $this->failure = array();
66 $credits = $this->getCredits();
67
68 $row = $this->articles_gateway->selectOneByIdAndUserId(
69 $this->input['id'],
70 $user_id
71 );
72
73 if ($row) {
74
75 // don't charge unless the article is ready
76 $decrement = false;
77
78 // is the article marked as ready?
79 if ($this->input['ready'] == 1) {
80
81 // did they offer at least the minimum?
82 if (
83 $credits > 0
84 && $this->input['credits_per_rating'] >= 0.01
85 && is_numeric($credits)
86 ) {
87
88 // was the article previously ready for review?
89 // (note 'row' not 'input')
90 if ($row['ready'] == 1) {
91
92 // only subtract (or add back) the difference to their
93 // account, since they already paid something
94 if (
95 is_numeric($row['credits_per_rating'])
96 && is_numeric($row['max_ratings'])
97 ) {
98 // user owes $credits, minus whatever they paid
99 // already
100 $amount = $row['credits_per_rating']
101 * $row['max_ratings']
102 $credits = $credits - $amount;
103 }
104
105 $decrement = true;
106
107 } else {
108 // article not ready previously, so they hadn't
109 // had credits deducted. if this is less than their
110 // in their account now, they may proceed.
111 $residual = $user->get('credits') - $credits;
112 $decrement = true;
113 }
114
115 } else {
116 $residual = -1;
117 $this->failure[] = "Credit offering invalid.";
118 $decrement = false;
119 }
120
121 } else {
122
123 // arbitrary positive value; they can proceed
124 $residual = 1;
125
126 // if it was previously ready but is no longer, refund them
127 if (
128 is_numeric($row['credits_per_rating'])
129 && is_numeric($row['max_ratings'])
130 && ($row['ready'] == 1)
131 ) {
132 // subtract a negative value
133 $amount = $row['credits_per_rating']
134 * $row['max_ratings']
135 $credits = -($amount);
136 $decrement = true;
137 }
138 }
139
140 if ($residual >= 0) {
141
142 $this->input['ip'] = $_SERVER['REMOTE_ADDR'];
143 $this->input['last_edited'] = $now;
144
145 if (! in_array(
146 $this->input['article_type'],
147 $this->article_types
148 )) {
149 $this->input['article_type'] = 1;
150 }
151
152 $result = $this->articles_gateway->updateByIdAndUserId(
153 $this->input['id'],
154 $user_id,
155 $this->input
156 );
157
158 if ($result) {
159 $article_id = $this->input['id'];
160
161 if ($decrement) {
162 $this->users_gateway->decrementCredits(
163 $user_id,
164 $credits
165 );
166 }
167 } else {
168 $this->failure[] = "Could not update article.";
169 }
170 } else {
171 $this->failure[] = "You do not have enough credits for ratings.";
172 }
173 }
174 }
175
176 public function submitNewArticle($user_id, $input)
177 {
178 $this->input = $this->filterInput($input);
179 $now = time();
180 $this->failure = array();
181 $credits = $this->getCredits();
182
183 $decrement = false;
184
185 // if the article is ready, we need to subtract credits.
186 if ($this->input['ready'] == 1) {
187
188 // if this is greater than or equal to 0, they may proceed.
189 if (
190 $credits > 0
191 && $this->input['credits_per_rating']>=0.01
192 && is_numeric($credits)
193 ) {
194 // minimum offering is 0.01
195 $residual = $user->get('credits') - $credits;
196 $decrement = true;
197 } else {
198 $residual = -1;
199 $this->failure[] = "Credit offering invalid.";
200 }
201
202 } else {
203 // arbitrary positive value if they are not done with their article.
204 // no deduction made yet.
205 $residual = 1;
206 }
207
208 // can user afford ratings on the new article?
209 if ($residual >= 0) {
210
211 // yes, insert the article
212 $this->input['last_edited'] = $now;
213 $this->input['ip'] = $_SERVER['REMOTE_ADDR'];
214 $article_id = $this->articles_gateway->insert($this->input);
215
216 if ($article_id) {
217 if ($decrement) {
218 // Charge them
219 $this->users_gateway->decrementCredits($user_id, $credits);
220 }
221 } else {
222 $this->failure[] = "Could not update credits.";
223 }
224
225 $result = $this->articles_gateway->updateByIdAndUserId(
226 $article_id,
227 $user_id,
228 $this->input
229 );
230
231 if (! $result) {
232 $this->failure[] = "Could not update article.";
233 }
234
235 } else {
236
237 // cannot afford ratings on new article
238 $this->failure[] = "You do not have enough credits for ratings.";
239 }
240 }
241 }
242 ?>
```
