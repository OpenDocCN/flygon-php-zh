# 附录 B. 门户网站之前的代码

本附录显示了一个遗留应用程序的部分页面脚本。它已经经过清理和匿名化处理，不是来自实际应用程序的。

这个脚本是系统的一部分，允许新闻学生撰写文章进行审查，并为其他学生的文章提供反馈。学生们可以为其他学生提供“积分”来审查他们的作品，并通过审查其他学生的文章来获得积分。由于积分是按审查支付的，学生们会限制最大审查次数，以确保他们不会用完积分。最后，他们被允许提供注释，指出审阅者应该注意的事项。

这是页面脚本在转换为使用网关类之前的版本。它只包含领域逻辑和数据源交互，而不包括初步设置或任何显示代码。

```php
1 <?php
2 // ...
3 require 'includes/setup.php';
4 // ...
5
6 $article_types = array(1, 2, 3, 4, 5);
7 $failure = array();
8 $now = time();
9
10 // sanitize and escape the user input
11 $input = $_POST;
12 $input['body'] = strip_tags($input['body']);
13 $input['notes'] = strip_tags($input['notes']);
14 foreach ($input as $key => $val) {
15 $input[$key] = mysql_real_escape_string($val);
16 }
17
18 if (isset($input['ready']) && $input['ready'] == 'on') {
19 $input['ready'] = 1;
20 } else {
21 $input['ready'] = 0;
22 }
23
24 // nothing less than 0.01 credits per rating
25 $input['credits_per_rating'] = round(
26 $input['credits_per_rating'],
27 2
28 );
29
30 $credits = round(
31 $input['credits_per_rating'] * $input['max_ratings'],
32 2
33 );
34
35 // updating an existing article?
36 if ($input['id']) {
37
38 // make sure this article belongs to the user
39 $stm = "SELECT *
40 FROM articles
41 WHERE user_id = '{$user_id}'
42 AND id = '{$input['id']}'
43 LIMIT 1";
44 $result = mysql_query($stm);
45
46 if (mysql_num_rows($result)) {
47
48 // get the existing article from the database
49 $row = mysql_fetch_assoc($result);
50
51 // don't charge unless the article is ready
52 $decrement = false;
53
54 // is the article marked as ready?
55 if ($input['ready'] == 1) {
56
57 // did they offer at least the minimum?
58 if (
59 $credits > 0
60 && $input['credits_per_rating'] >= 0.01
61 && is_numeric($credits)
62 ) {
63
64 // was the article previously ready for review?
65 // (note 'row' not 'input')
66 if ($row['ready'] == 1) {
67
68 // only subtract (or add back) the difference to their
69 // account, since they already paid something
70 if (
71 is_numeric($row['credits_per_rating'])
72 && is_numeric($row['max_ratings'])
73 ) {
74 // user owes $credits, minus whatever they paid already
75 $amount = $row['credits_per_rating']
76 * $row['max_ratings']
77 $credits = $credits - $amount;
78 }
79
80 $decrement = true;
81
82 } else {
83 // article not ready previously, so they hadn't
84 // had credits deducted. if this is less than their
85 // in their account now, they may proceed.
86 $residual = $user->get('credits') - $credits;
87 $decrement = true;
88 }
89
90 } else {
91 $residual = -1;
92 $failure[] = "Credit offering invalid.";
93 $decrement = false;
94 }
95
96 } else {
97
98 // arbitrary positive value; they can proceed
99 $residual = 1;
100
101 // if it was previously ready but is no longer, refund them
102 if (
103 is_numeric($row['credits_per_rating'])
104 && is_numeric($row['max_ratings'])
105 && ($row['ready'] == 1)
106 ) {
107 // subtract a negative value
108 $amount = $row['credits_per_rating']
109 * $row['max_ratings']
110 $credits = -($amount);
111 $decrement = true;
112 }
113 }
114
115 if ($residual >= 0) {
116
117 if (strlen($input['notes'])>0) {
118 $notes = "notes = '{$input['notes']}'";
119 } else {
120 $notes = "notes = NULL";
121 }
122
123 if (strlen($input['title'])>0) {
124 $title = "title = '{$input['title']}'";
125 } else {
126 $title = "title = NULL";
127 }
128
129 if (! in_array(
130 $input['article_type'],
131 $article_types
132 )) {
133 $input['article_type'] = 1;
134 }
135
136 $stm = "UPDATE articles
137 SET
138 body = '{$input['body']}',
139 $notes,
140 $title,
141 article_type = '{$input['article_type']}',
142 ready = '{$input['ready']}',
143 last_edited = '{$now}',
144 ip = '{$_SERVER['REMOTE_ADDR']}',
145 credits_per_rating = '{$input['credits_per_rating']}',
146 max_ratings = '{$input['max_ratings']}'
147 WHERE user_id = '{$user_id}'
148 AND id = '{$input['id']}'";
149
150 if (mysql_query($stm)) {
151 $article_id = $input['id'];
152
153 if ($decrement) {
154 // Charge them
155 $stm = "UPDATE users
156 SET credits = credits - {$credits}
157 WHERE user_id = '{$user_id}'";
158 mysql_query($stm);
159 }
160 } else {
161 $failure[] = "Could not update article.";
162 }
163 } else {
164 $failure[] = "You do not have enough credits for ratings.";
165 }
166 }
167
168 } else {
169
170 // creating a new article. do not decrement until specified.
171 $decrement = false;
172
173 // if the article is ready, we need to subtract credits.
174 if ($input['ready'] == 1) {
175
176 // if this is greater than or equal to 0, they may proceed.
177 if (
178 $credits > 0
179 && $input['credits_per_rating']>=0.01
180 && is_numeric($credits)
181 ) {
182 // minimum offering is 0.01
183 $residual = $user->get('credits') - $credits;
184 $decrement = true;
185 } else {
186 $residual = -1;
187 $failure[] = "Credit offering invalid.";
188 }
189
190 } else {
191 // arbitrary positive value if they are not done with their article.
192 // no deduction made yet.
193 $residual = 1;
194 }
195
196 // can user afford ratings on the new article?
197 if ($residual >= 0) {
198
199 // yes, insert the article
200 $stm = "INSERT INTO articles (
201 user_id,
202 ip,
203 last_edited,
204 article_type
205 ) VALUES (
206 '{$user_id}',
207 '{$_SERVER['REMOTE_ADDR']}',
208 '$now',
209 '$input['article_type']'
210 )";
211
212 if (mysql_query($stm)) {
213 $article_id = mysql_insert_id();
214 if ($decrement) {
215 // Charge them
216 $stm = "UPDATE users
217 SET credits = credits - {$credits}
218 WHERE user_id='{$user_id}'";
219 mysql_query($stm);
220 }
221 } else {
222 $failure[] = "Could not update credits.";
223 }
224
225 $stm = "UPDATE articles
226 SET
227 body = '{$input['body']}',
228 $notes,
229 $title,
230 article_type = '{$input['article_type']}',
231 ready = '{$input['ready']}',
232 last_edited = '$now',
233 ip = '{$_SERVER['REMOTE_ADDR']}',
234 credits_per_rating = '{$input['credits_per_rating']}',
235 max_ratings = '{$input['max_ratings']}'
236 WHERE
237 user_id = '{$user_id}'
238 AND id = '$article_id'
239 ";
240
241 if (! mysql_query($stm)) {
242 $failure[] = "Could not update article.";
243 }
244
245 } else {
246
247 // cannot afford ratings on new article
248 $failure[] = "You do not have enough credits for ratings.";
249 }
250 }
251 ?>
```
