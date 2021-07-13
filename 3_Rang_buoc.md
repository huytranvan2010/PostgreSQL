# Database constraint
Ý tưởng của cơ sở dữ liệu là sắp xếp dữ liệu theo cấu trúc nhất định - một mô hình được xác định trước, nơi chúng ta sử dụng các kiểu dữ liệu, mối quan hệ và quy tắc lên cấu trúc đó. Nói chung những qu tắc này được gọi là các **ràng buộc toàn vẹn (integrity constraints)**.
Các ràng buộc toàn vẹn có thể được chia thành 3 loại:
* Ràng buộc thuộc tính, ví dụ kiểu dữ liệu cột
* Ràng buộc khóa, ví dụ khóa chính
* Ràng buộc tham chiếu được thực hiện thông qua các khóa ngoại

**Tại sao sử dụng các ràng buộc?**
* Các ràng buộc định hình cấu trúc dữ liệu. Nếu dữ liệu được nhập dài dòng và không theo nguyên tắc sẽ dẫn đến khó khăn trong quá trình xử lý.
* Các ràng buộc mang đến tính nhất quán cho dữ liệu, có nghĩa ràng 1 hàng trong một bảng nhất định có cấu trúc giống như hàng tiếp theo...

Đối với một bảng đã tồn tại, ta có thể sử dụng lệnh `ADD CONSTRAINT` để thực hiện ràng buộc như sau:
```SQL
ALTER TABLE table_name
ADD CONSTRAINT constraint_name constaint_definition;
```

## 1. Kiểu dữ liệu
Chúng ta đã tìm hiểu 4 loại dữ liệu chính trong PostgreSQL. Đối với những **ràng buộc thuộc tính** là các loại dữ liệu thì có thể được chỉ định cho từng cột trong bảng.
**Các kiểu dữ liệu là các ràng buộc thuộc tính và được triển khai trên các cột của bảng.** Chúng định nghĩa miền giá trị của mỗi cột. Thông qua việc này dữ liệu được lưu trữ một cách nhất quán. Như vậy chất lượng dữ liệu được nâng cao.

### 1.1. Ràng buộc thuộc tính đặc biệt
Có 2 thuộc tính đặc biệt: các ràng buộc `NOT NULL` và `UNIQUE`.
#### a. NULL
**Giá trị `NULL` nghĩa là một giá trị không được xác định hoặc hoàn toàn không tồn tại**. Ràng buộc `NOT NULL` không cho phép có giá trị (bị) `NULL` trong một cột nhất định. Điều này đúng với trạng thái hiện tại của cơ sở dữ liệu nhưng cũng đúng với bất kì trạng thái nào trong tương lai. Chúng ta chỉ có thể chỉ định ràng buộc `NOT NULL` trên cột không có giá trị `NULL` và không thể chèn các giá trị `NULL` vào cột đấy trong tương lai sau khi đã ràng buộc thuộc tính `NOT NULL`.

Ví dụ chúng ta tạo một bảng như sau:
```SQL
CREATE TABLE students (
    ssn INTEGER NOT NULL,
    lastname VARCHAR(64) NOT NULL,
    home_phone INTEGER
);
```
Hai cột đầu tiên là `ssn` và `lastname` được đặt ràng buộc là `NOT NULL`, nghĩa là bất kì bản ghi nào trong bảng students cũng phải có sẵn `ssn` và `lastname`. Cột `home_phone` có thể chứa giá trị `NULL`, đây là mặc định (do nó không ghi gì).
Tuy nhiên chúng ta cũng có thể thêm ràng buộc `NOT NULL` đối với cột `home_phone` cho bảng `students` như sau:
```SQL
ALTER TABLE students ALTER COLUMN home_phone SET NOT NULL;
```

#### b. UNIQUE
Ràng buộc `UNIQUE` trên cột đảm bảo rằng không có sự trùng lặp giá trị trong cột đó. Cũng giống như ràng buộc `NOT NULL`, chúng ta chỉ có thể thêm một ràng buộc `UNIQUE` nếu cột không tồn tại sự trùng lặp nào trước khi bạn đặt ràng buộc lên cột (phải thỏa mãn trạng thái hiện tại). Dưới đây là cú pháp:
```SQL
CREATE TABLE tên_bảng (
    tên_cột kiểu_dữ_liệu UNIQUE
);
```
Hoặc cho bảng đã tồn tại như sau:
```SQL
ALTER TABLE tên_bảng ADD CONSTRAINT tên_ràng_buộc UNIQUE(tên_cột);
```

## Khóa
Bảng trong cơ sở dữ liệu có một thuộc tính hoặc kết hợp nhiều thuộc tính mà giá trị của chungs là giá trị duy nhất trên toàn bộ bảng. Các thuộc tính như vậy xác định một bản ghi duy nhất (ví dụ ID, số thứ tự...).

Một bảng chỉ chứa các bản ghi khác biệt (tính duy nhất của các bản ghi), có nghĩa là sự kết hợp của tất cả các thuộc tính là một khóa trong chính nó (và các bản ghi khác nhau mà). Tuy nhiên nó vẫn chưa được gọi là khóa chính mà được gọi là siêu khóa (superkey). Nếu xóa một hoặc một số thuộc tính trong tập hợp các thuộc tính đó cho đến khi không thể xóa được nữa mà tập hợp thuộc tính đó vẫn xác định tính duy nhất của các bản ghi đó thì tập hợp đó là khóa

### Khóa chính (primary key)
**Khóa chính** là một trong những khái niệm quan trọng nhất trong thiết kế cơ sở dữ liệu.
* Hầu hết mọi bảng cơ sở dữ liệu đều có một khóa chính. Mục đích chính của khóa chính là xác định tính duy nhất của các bản ghi trong một bảng.
* Các khóa chính cần được xác định trên các cột không chấp nhận giá trị trùng lặp hoặc NULL.
* Các ràng buộc khóa chính là bất biến theo thời gian. Do đó nên chọn các cột trong đó các giá trị luôn là duy nhất và không rỗng.

Thông thường chúng ta thêm khóa chính vào bảng khi xác định cấu trúc bảng:
```SQL
CREATE TABLE TABLE (
    cột_1 kiểu_dữ_liệu PRIMARY KEY,
    cột_2 kiểu_dữ_liệu,
);
```

Trong trường hợp khóa chính bao gồm hai hay nhiều cột, chúng ta cần xác định ràng buộc khóa chính như sau:
```SQL
CREATE TABLE (
    cột_1 kiểu_dữ_liệu,
    cột_2 kiểu_dữ_liệu,
    ...
    PRIMARY KEY (cột_1, cột_2)
);
```

Thêm ràng buộc `PRIMARY KEY` vào bảng hiện có là quy trình tương tự như thêm ràng buộc `UNIQUE` mà chúng ta đã làm quen từ trước:
```SQL
ALTER TABLE tên_bảng
ADD CONSTRAINT tên_ràng_buộc PRIMARY KEY(tên_cột);
```
Xem ví dụ sau đây về tạo bảng `bicycles`:
```SQL
CREATE TABLE bicycles (
    id INTEGER NOT NULL,
    make VARCHAR(50) NOT NULL,
    model VARCHAR(50) NOT NULL,
)
```
Chúng ta sẽ sửa cột `id` thành khóa chính và đặt tên ràng buộc là `id_pk` như sau:
```SQL
ALTER TABLE
ADD CONSTRAINT id_pk PRIMARY KEY(id)
```


### Khóa ngoại
#### Khóa ngoại
Mối quan hệ giữa các bảng được thể hiện bằng các `khóa ngoại` - các cột của bảng này được chỉ định trỏ đến `khóa chính` của bảng khác.

Có một số quy định đối với khóa ngoại:
* Đầu tiên tên và kiểu dữ liệu phải giống với `khóa chính` được trỏ đến
* Thứ hai chỉ các giá trị `khóa ngoại` được phép tồn tại dưới dạng giá trị trong `khóa chính` của bảng được tham chiếu. Đây là ràng buộc khóa ngoại, còn được gọi là tính toàn vẹn tham chiếu (referential integrity)
* Cuối cùng **khóa ngoại** không nhất thiết phải là khóa vì các giá trị trùng lặp và giá trị `NULL` được cho phép đối với cột khóa ngoại.

Để xác định `khóa ngoại` chúng ta sử dụng một trong các cách sau:
```SQL
-- cách 1
CREATE TABLE bảng_con(
    c1 kiểu_dữ_liệu PRIMARY KEY,
    c2 kiểu_dữ_liệu REFERENCES bảng_bố(p2)
);
-- cách 2
CREATE TABLE bảng_con(
    c1 kiểu_dữ_liệu PRIMARY KEY,
    c2 kiểu_dữ_liệu,
    FOREIGN KEY (c2) REFERENCES bảng_bố(p2)
);
```

trong trường hợp khóa ngoại là một nhóm cột, chúng ta có thể xác định ràng buộc khóa ngoại bằng cú pháp sau:
```SQL
CREATE TABLE bảng_con (
    c1 kiểu_dữ_liệu PRIMARY KEY,
    c2 kiểu_dữ_liệu,
    c3 kiểu_dữ_liệu,
    FOREIGN KEY (c2, c3) REFERENCES bảng_bố(p2, p3)
);
```

Cú pháp để thêm khóa ngoại vào bảng đã được tạo:
```SQL
ALTER TABLE a
ADD COLUMN CONSTRAINT a_fkey FOREIGN KEY(b_id) REFERENCES b(id);
```

#### Tính toàn vẹn tham chiếu
Tính toàn vẹn tham chiếu - một bản ghi tham chiếu đến một bảng khác phải luôn luôn tham chiếu đến một bản ghi hiện có. Nói cách khác một bản ghi trong bảng `A` không thể trỏ đến một bản ghi *không tồn tại* trong bảng `B`. Tính toàn vẹn của tham chiếu là một ràng buộc luôn liên quan đến hai bảng và được thi hành thông qua các `khóa ngoại`.

Tính toàn vẹn tham chiếu có thể bị vi phạm bởi 2 cách:
* Giả sử bảng `A` tham chiếu bảng `B`. Nếu một bản ghi trong `B` bị xóa sẽ vi phạm tính toàn vẹn tham chiếu.
* Nếu cố gắng chèn một bản ghi vào bảng `A` nhưng nó không tồn tại trong bảng `B` cũng bi phạm nguyên tắc.

Đó là những lý do chính `khóa ngoại` gây lỗi và ngăn chúng ta thực hiện điều này. Ngoài thông báo lỗi chúng ra có thể cho hệ thống biết điều gì xảy ra nếu một mục trong bảng được tham chiếu bị xóa. Theo mặc định từ khóa `ON DELETE NO ACTION` được tự động gán vào định nghĩa `khóa ngoại`:
```SQL
CREATE TABLE a (
    id INTEGER PRIMARY KEY,
    column_a VARCHAR(50),
    ...
    b_id INTEGER REFERENCES b (id) ON DELETE NO ACTION
);
```

Điều này có nghĩa rằng nếu cố gắng xóa một bản ghi trong bảng `B` được tham chiếu từ `A`, hệ thống sẽ đưa ra lỗi. Tuy nhiên còn có tùy chọn khác. Ví dụ có tùy chọn `CASCADE`, trươc tiên sẽ xóa bản ghi trong bảng `B`, sau đó sẽ tự đống xóa tất cả các bản ghi tham chiếu trong bảng A. Vì vậy việc xóa đó được xếp tầng (CASCADE).
```SQL
CREATE TABLE a (
    id INTEGER PRIMARY KEY,
    column_a VARCHAR(50),
    ...,
    b_id INTEGER REFERENCES b (id) ON DELETE CASCADE
);
```

## Ràng buộc Check
Ràng buộc `CHECK` để kiểm tra giá trị trong cột phải đáp ứng một yêu cầu nhất định. Ràng buộc `CHECK` sử dụng biểu thức Boolean để đánh giá các giá trị trước khi chèn hoặc cập nhật vào cột. Nếu các giá trị vượt qua kiểm tra PostgreSQL sẽ chèn haowcj cập nhật các giá trị này vào cột.

Xác định ràng buộc `CHECK` khi tạo bảng mới trong PostgreSQL như sau:
```SQL
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birth_date DATE CHECK (birht_data > '1990-01-01'),
    joined_date DATE CHECK (joined_date > birth_date),
    salary NUMERIC CHECK (salary > 0)
);
```

Bảng `employees` có 3 ràng chuộc `CHECK` như sau:
* Ngày sinh (birth_date) của nhân viên lớn hơn `01/01/1990`.
* Ngày tham gia (joined_date) phải lớn hơn ngày sinh (birth_date).
* Mức lương phải lớn hơn 0

Thử chèn một hàng mới vào bảng `employees` như sau:
```SQL
INSERT INTO employees (first_name, last_name, birth_date, joined_date, salary) 
VALUES ('John', 'Doe', '1972-01-01', '2015-07-01', -1300);
```

Câu lệnh trên chèn mức lương âm vào cột `salary` do đó PostgreSQL báo lỗi như sau:
```SQL
ERROR: new row for relation "employees" violates check constraint "employees_salary_check"
```

Xác định ràng buộc `CHECK` trong các bảng hiện có:
```SQL
CREATE TABLE prices_list (
    id SERIAL PRIMARY KEY,
    product_id INTEGER NOT NULL,
    price NUMERIC NOT NULL,
    discount NUMERIC NOT NULL,
    valid_from DATE NOT NULL,
    valid_to DATE NOT NULL
);
```

Bây giờ sử dụng câu lệnh `ALTER TABLE` để thêm các ràng buộc `CHECK` vào bảng `prices_list` rằng `price` và `discount` phải lớn hơn 0 và `discount` nhỏ hơn `price`. Như vậy chúng ta cần sử dụng biểu thức Boolean chứa toán tử `AND` để thêm ràng buộc như sau:
```SQL
ALTER TABLE prices_list ADD CONSTRAINT price_discount_check CHECK (price > 0 AND discount >= 0 AND price > discount)
```

Xem thêm một ví dụ khác về ràng buộc `CHECK`:
```SQL
CREATE TABLE employees (
    id SMALLINT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birth_date DATE CHECK,
    joined_date DATE CHECK,
    salary NUMERIC
);

--Add constraint for birth_date column
ALTER TABLE employees ADD CONSTRAINT CHK_Birthdate CHECK ( birth_date > '1900-01-01' );
--Add constraint for joined_date column
ALTER TABLE employees ADD CONSTRAINT CHK_JoinedDate CHECK ( joined_date > birth_date );
--Add constraint for salary column
ALTER TABLE employees ADD CONSTRAINT CHK_Salary CHECK ( salary > 0 );
```

## Loại bỏ ràng buộc
Câu lệnh `DROP CONSTRAINT` dùng để xóa các ràng buộc `UNIQUE, PRIMARY KEY, FOREIGN KEY, CHECK`.
Cú pháp để loại bỏ ràng buộc trong PostgreSQL như sau:
```SQL
ALTER TABLE tên_bảng DROP CONSTRAINT tên_ràng_buộc;
```

Cú pháp để loại bỏ nhiều ràng buộc trong một bảng là:
```SQL
ALTER TABLE tên_bảng
DROP CONSTRAINT tên_ràng_buộc,
DROP CONSTRAINT tên_ràng_buộc,
...;
```

Thêm ví dụ về xóa ràng buộc:
```SQL
--Define these contraints here
ALTER TABLE employees
ADD CONSTRAINT chk_birthdate CHECK ( birth_date < '1900-01-01' ),
ADD CONSTRAINT chk_joineddate CHECK ( joined_date < birth_date ),
ADD CONSTRAINT chk_salary CHECK ( salary < 0 );

--Drop these constraints here
ALTER TABLE employees
DROP CONSTRAINT chk_birthdate,
DROP CONSTRAINT chk_joineddate,
DROP CONSTRAINT chk_salary;

-- Insert the information of employees to databse
INSERT INTO employees (id, first_name, last_name, birth_date, joined_date, salary) 
VALUES
('1','First','First','1901-01-01','1950-01-01',1000),
('2','Second','Second','1902-02-02','1950-01-01',1000),
('3','Third','Third','1903-03-03','1950-01-01',1000);

--Show the information of all employees
SELECT * FROM employees;
```



