# 附录 E. 收集演示逻辑之前的代码

```php
**articles.php**
1 <?php
2 require "includes/setup.php";
3
4 $current_page = 'articles';
5
6 include "header.php";
7
8 $id = isset($_GET['id']) ? $_GET['id'] : 0;
9 if ($id) {
10 $page_title = "Edit An Article";
11 } else {
12 $page_title = "Submit An Article";
13 }
14
15 ?><h1><?php echo $page_title ?></h1><?php
16
17 $user_id = $user->getId();
18
19 $db = new Database($db_host, $db_user, $db_pass);
20 $articles_gateway = new ArticlesGateway($db);
21 $users_gateway = new UsersGateway($db);
22 $article_transactions = new ArticleTransactions(
23 $articles_gateway,
24 $users_gateway
25 );
26
27 if ($id) {
28 $article_transactions->updateExistingArticle($user_id, $_POST);
29 } else {
30 $article_transactions->submitNewArticle($user_id, $_POST);
31 }
32
33 $failure = $article_transactions->getFailure();
34 $input = $article_transactions->getInput();
35
36 ?>
37
38 <?php
39 if ($failure) {
40 $failure_text = implode("<br />\n", $failure);
41 echo "<h2>Failure</h2>";
42 echo "<p>We could not save the article.<br />";
43 echo $failure_text. "</p>";
44 } else {
45 echo "
46 <h2>Success</h2>
47 <p>We saved the article.</p>
48 ";
49 }
50 ?>
51
52 <form method="POST" action="<?php echo $_SERVER['PHP_SELF']?>">
53
54 <input type="hidden" name="id" value="<?php echo $id ?>" />
55
56 <h3>Title</h3>
57 <input type="text" name="title" value="<?php
58 echo $input['title']
59 ?>" size="100">
60
61
62 <h3>Article</h3>
63 <textarea name="body" cols="80" rows="30"><?php
64 echo stripslashes($input['body'])
65 ?></textarea>
66
67 <h3>Ratings</h3>
68 <p>How many rated reviews do you want?</p>
69 <select name='max_ratings'>
70 <?php for ($i = 1; $i <= 10; $i ++) {
71 echo "<option value='$i' ";
72 if ($input['max_ratings'] == $i) {
73 echo 'selected="selected"';
74 }
75 echo ">$i</option>\n";
76 } ?>
77 </select>
78
79 <p>How many credits will you give for each rating?</p>
80 <input type='text' name='credits_per_rating' value='<?php
81 echo $input['credits_per_rating'];
82 ?>' size='5' />
83
84 <h3>Notes for Reviewers</h3>
85 <input type="text" name="notes" value="<?php
86 echo $input['notes']
87 ?>" size="100">
88 <label><input type="checkbox" name="ready" <?php
89 echo $input['ready'] ? 'checked="checked"' : '';
90 ?> /> This article is ready to be rated.</label>
91
92 <p align="center">
93 <input type="submit" value="Save" name="submit">
94 </p>
95
96 </form>
97
98 <?php
99 include "footer.php";
100 ?>
```
