
**1. Kết nối dữ liệu database:
------------------------------

PHP cung cấp hàm mysql_connect để mở một kết nối với Database. Hàm này nhận 5 tham số và trả về một định danh liên kết với MySQL nếu thành công hoặc FALSE nếu thất bại.

connection `mysql_connect(server,user,passwd,new_link,client_flag);`

ví dụ: 

```sh
<?php
   
   $dbhost = 'localhost:3036';
   $dbuser = 'tennguoidung';
   $dbpass = 'matkhau';
   $conn = mysql_connect($dbhost, $dbuser, $dbpass);
   
   if(! $conn )
   {
      die('Không thể kết nối: ' . mysql_error());
   }
   
   echo 'Kết nối thành công';
   mysql_close($conn);
?>
```

**2. Trang đăng nhập đăng kí có kết nối database:
-------------------------------------------------

- Đầu tiền tạo một file php làm chức năng kết nối dữ liệu để tiện cho việc gọi ra:

```sh
<?php
    $ketnoi['host'] = 'localhost'; //Tên server, nếu dùng hosting free thì cần thay đổi
    $ketnoi['dbname'] = 'dangki'; //Đây là tên của Database
    $ketnoi['username'] = 'tanthienqn'; //Tên sử dụng Database
    $ketnoi['password'] = '';//Mật khẩu của tên sử dụng Database
    @mysql_connect(
        "{$ketnoi['host']}",
        "{$ketnoi['username']}",
        "{$ketnoi['password']}")
    or
        die("Không thể kết nối database");
    @mysql_select_db(
        "{$ketnoi['dbname']}") 
    or
        die("Không thể chọn database");
?>
```

- Xây dựng chức năng đăng ký thành viên:

SQL:

```sh
CREATE TABLE member (
        username NVARCHAR(32) NOT NULL,
        password NVARCHAR(16) NOT NULL,
        email NVARCHAR(50) NOT NULL,
        fullname NVARCHAR(50) NOT NULL,
        birthday NVARCHAR(10) NOT NULL,
        sex NVARCHAR(3) NOT NULL
    )
```

- Tạo trang đăng kí: `dangki.php`

```sh
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <title>Trang đăng lý</title>
    </head>
    <body>
        <h1>Trang đăng ký thành viên</h1>
        <form action="xuly.php" method="POST">
            <table cellpadding="0" cellspacing="0" border="1">
                <tr>
                    <td>
                        Tên đăng nhập : 
                    </td>
                    <td>
                        <input type="text" name="txtUsername" size="50" />
                    </td>
                </tr>
                <tr>
                    <td>
                        Mật khẩu :
                    </td>
                    <td>
                        <input type="password" name="txtPassword" size="50" />
                    </td>
                </tr>
                <tr>
                    <td>
                        Email :
                    </td>
                    <td>
                        <input type="text" name="txtEmail" size="50" />
                    </td>
                </tr>
                <tr>
                    <td>
                        Họ và tên :
                    </td>
                    <td>
                        <input type="text" name="txtFullname" size="50" />
                    </td>
                </tr>
                <tr>
                    <td>
                        Ngày sinh :
                    </td>
                    <td>
                        <input type="text" name="txtBirthday" size="50" />
                    </td>
                </tr>
                <tr>
                    <td>
                        Giới tính :
                    </td>
                    <td>
                        <select name="txtSex">
                            <option value=""></option>
                            <option value="Nam">Nam</option>
                            <option value="Nu">Nữ</option>
                        </select>
                    </td>
                </tr>
            </table>
            <input type="submit" value="Đăng ký" />
            <input type="reset" value="Nhập lại" />
        </form>
    </body>
</html>
```

- Tạo một trang xử lý, nhận thông tin rồi lưu vào sql: `xuly.php`:

```sh
<?php
 
    // Nếu không phải là sự kiện đăng ký thì không xử lý
    if (!isset($_POST['txtUsername'])){
        die('');
    }
     
    //Nhúng file kết nối với database
    include('ketnoi.php');
          
    //Khai báo utf-8 để hiển thị được tiếng việt
    header('Content-Type: text/html; charset=UTF-8');
          
    //Lấy dữ liệu từ file dangky.php
    $username   = addslashes($_POST['txtUsername']);
    $password   = addslashes($_POST['txtPassword']);
    $email      = addslashes($_POST['txtEmail']);
    $fullname   = addslashes($_POST['txtFullname']);
    $birthday   = addslashes($_POST['txtBirthday']);
    $sex        = addslashes($_POST['txtSex']);
          
    //Kiểm tra người dùng đã nhập liệu đầy đủ chưa
    if (!$username || !$password || !$email || !$fullname || !$birthday || !$sex)
    {
        echo "Vui lòng nhập đầy đủ thông tin. <a href='javascript: history.go(-1)'>Trở lại</a>";
        exit;
    }
          
        // Mã khóa mật khẩu
        $password = md5($password);
          
    //Kiểm tra tên đăng nhập này đã có người dùng chưa
    if (mysql_num_rows(mysql_query("SELECT username FROM member WHERE username='$username'")) > 0){
        echo "Tên đăng nhập này đã có người dùng. Vui lòng chọn tên đăng nhập khác. <a href='javascript: history.go(-1)'>Trở lại</a>";
        exit;
    }
                    
    //Lưu thông tin thành viên vào bảng
    @$addmember = mysql_query("
        INSERT INTO member (
            username,
            password,
            email,
            fullname,
            birthday,
            sex
        )
        VALUE (
            '{$username}',
            '{$password}',
            '{$email}',
            '{$fullname}',
            '{$birthday}',
            '{$sex}'
        )
    ");
                          
    //Thông báo quá trình lưu
    if ($addmember)
        echo "Quá trình đăng ký thành công. <a href='/'>Về trang chủ</a>";
    else
        echo "Có lỗi xảy ra trong quá trình đăng ký. <a href='dangky.php'>Thử lại</a>";
?>
```

- Xây dựng chức năng đăng nhập:

Trang đăng nhập:

```sh
<?php
//Khai báo sử dụng session
session_start();
 
//Khai báo utf-8 để hiển thị được tiếng việt
header('Content-Type: text/html; charset=UTF-8');
 
//Xử lý đăng nhập
if (isset($_POST['dangnhap'])) 
{
    //Kết nối tới database
    include('ketnoi.php');
     
    //Lấy dữ liệu nhập vào
    $username = addslashes($_POST['txtUsername']);
    $password = addslashes($_POST['txtPassword']);
     
    //Kiểm tra đã nhập đủ tên đăng nhập với mật khẩu chưa
    if (!$username || !$password) {
        echo "Vui lòng nhập đầy đủ tên đăng nhập và mật khẩu. <a href='javascript: history.go(-1)'>Trở lại</a>";
        exit;
    }
     
    // mã hóa pasword
    $password = md5($password);
     
    //Kiểm tra tên đăng nhập có tồn tại không
    $query = mysql_query("SELECT username, password FROM member WHERE username='$username'");
    if (mysql_num_rows($query) == 0) {
        echo "Tên đăng nhập này không tồn tại. Vui lòng kiểm tra lại. <a href='javascript: history.go(-1)'>Trở lại</a>";
        exit;
    }
     
    //Lấy mật khẩu trong database ra
    $row = mysql_fetch_array($query);
     
    //So sánh 2 mật khẩu có trùng khớp hay không
    if ($password != $row['password']) {
        echo "Mật khẩu không đúng. Vui lòng nhập lại. <a href='javascript: history.go(-1)'>Trở lại</a>";
        exit;
    }
     
    //Lưu tên đăng nhập
    $_SESSION['username'] = $username;
    echo "Xin chào " . $username . ". Bạn đã đăng nhập thành công. <a href='/'>Về trang chủ</a>";
    die();
}
?>
<!DOCTYPE html>
<html>
    <head>
        <title></title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
        <form action='dangnhap.php?do=login' method='POST'>
            <table cellpadding='0' cellspacing='0' border='1'>
                <tr>
                    <td>
                        Tên đăng nhập :
                    </td>
                    <td>
                        <input type='text' name='txtUsername' />
                    </td>
                </tr>
                <tr>
                    <td>
                        Mật khẩu :
                    </td>
                    <td>
                        <input type='password' name='txtPassword' />
                    </td>
                </tr>
            </table>
            <input type='submit' name="dangnhap" value='Đăng nhập' />
            <a href='dangky.php' title='Đăng ký'>Đăng ký</a>
        </form>
    </body>
</html>
```

- Trang chủ để thực hiện đăng kí và đăng nhập: `trangchu.php`

```sh
<?php session_start(); ?>
<!DOCTYPE html>
<html>
    <head>
        <title></title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
       <?php 
       if (isset($_SESSION['txtUsername']) && $_SESSION['txtUsername']){
           echo 'Bạn đã đăng nhập với tên là '.$_SESSION['txtUsername']."<br/>";
           echo 'Click vào đây để <a href="logout.php">Logout</a>';
       }
       else{
           echo 'Bạn chưa đăng nhập';
       }
       ?>
    </body>
</html>
```

**2. Chức năng tìm kiếm:
------------------------

gọi hàm `ketnoi.php` đã xây dựng ở trên để kết nối database

```sh
<html>
    <head>
        <title>Chương trình tìm kiếm</title>
    </head>
    <body>
        <div align="center">
            <form action="search.php" method="get">
                Search: <input type="text" name="search" />
                <input type="submit" name="ok" value="search" />
            </form>
        </div>
        <?php
        if (isset($_REQUEST['ok'])) 
        {
            $search = addslashes($_GET['search']);
 
            if (empty($search)) {
                echo "Yeu cau nhap du lieu vao o trong";
            } 
            else
            {
                $query = "select * from users where username like '%$search%'";
 
                include('ketnoi.php');
 
                $sql = mysql_query($query);
 
                $num = mysql_num_rows($sql);
                if ($num > 0 && $search != "") 
                {
                    echo "$num ket qua tra ve voi tu khoa <b>$search</b>";
                    echo '<table border="1" cellspacing="0" cellpadding="10">';
                    while ($row = mysql_fetch_assoc($sql)) {
                        echo '<tr>';
                            echo "<td>{$row['txtUsername']}</td>";
                            echo "<td>{$row['txtEmail']}</td>";
                            echo "<td>{$row['txtFullname']}</td>";
                            echo "<td>{$row['txtBirthday']}</td>";
                            echo "<td>{$row['txtSex']}</td>";
                        echo '</tr>';
                    }
                    echo '</table>';
                } 
                else {
                    echo "Khong tim thay ket qua!";
                }
            }
        }
        ?>   
    </body>
</html>
```

