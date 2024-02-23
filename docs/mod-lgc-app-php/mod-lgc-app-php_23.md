# 附录 F. 收集演示逻辑后的代码

```php
**articles.php**
1 <?php
2 require "includes/setup.php";
3
4 $user_id = $user->getId();
5
6 $db = new Database($db_host, $db_user, $db_pass);
7 $articles_gateway = new ArticlesGateway($db);
8 $users_gateway = new UsersGateway($db);
9 $article_transactions = new ArticleTransactions(
10 $articles_gateway,
11 $users_gateway
12 );
13
14 $id = isset($_GET['id']) ? $_GET['id'] : 0;
15 if ($id) {
16 $article_transactions->updateExistingArticle($user_id, $_POST);
17 } else {
18 $article_transactions->submitNewArticle($user_id, $_POST);
19 }
20
21 $failure = $article_transactions->getFailure();
22 $input = $article_transactions->getInput();
23 $action = $_SERVER['PHP_SELF'];
24
25 /** PRESENTATION */
26
27 $current_page = 'articles';
28
29 include "header.php";
30
31 if ($id) {
32 $page_title = "Edit An Article";
33 } else {
34 $page_title = "Submit An Article";
35 }
36 ?>
37
38 <h1><?php echo $page_title ?></h1><?php
39
40 if ($failure) {
41 $failure_text = implode("<br />\n", $failure);
42 echo "<h2>Failure</h2>";
43 echo "<p>We could not save the article.<br />";
44 echo $failure_text. "</p>";
45 } else {
46 echo "
47 <h2>Success</h2>
48 <p>We saved the article.</p>
49 ";
50 }
51 ?>
52
53 <form method="POST" action="<?php echo $action ?>">
54
55 <input type="hidden" name="id" value="<?php echo $id ?>" />
56
57 <h3>Title</h3>
58 <input type="text" name="title" value="<?php
59 echo $input['title']
60 ?>" size="100">
61
62
63 <h3>Article</h3>
64 <textarea name="body" cols="80" rows="30"><?php
65 echo stripslashes($input['body'])
66 ?></textarea>
67
68 <h3>Ratings</h3>
69 <p>How many rated reviews do you want?</p>
70 <select name='max_ratings'>
71 <?php for ($i = 1; $i <= 10; $i ++) {
72 echo "<option value='$i' ";
73 if ($input['max_ratings'] == $i) {
74 echo 'selected="selected"';
75 }
76 echo ">$i</option>\n";
77 } ?>
78 </select>
79
80 <p>How many credits will you give for each rating?</p>
81 <input type='text' name='credits_per_rating' value='<?php
82 echo $input['credits_per_rating'];
83 ?>' size='5' />
84
85 <h3>Notes for Reviewers</h3>
86 <input type="text" name="notes" value="<?php
87 echo $input['notes']
88 ?>" size="100">
89 <label><input type="checkbox" name="ready" <?php
90 echo $input['ready'] ? 'checked="checked"' : '';
91 ?> /> This article is ready to be rated.</label>
92
93 <p align="center">
94 <input type="submit" value="Save" name="submit">
95 </p>
96
97 </form>
98
99 <?php
100 include "footer.php";
101 ?>
```
