---
tags:
  - プログラミング
  - CSharp
  - データベース
---
# データベースとテーブルの作成

SQL Server Management Studioから行う

# CSharpからSQLServerにつなぐ方法

SQLServerにつなぐにはサーバー名と認証方式が必要
SQLServer認証はユーザー名とパスワードが必要

# サンプルコード

## エンティティー

```c#
namespace Hoimi.SqlServer
{
    public sealed class ProductEntity
    {
        // 完全コンストラクタパターン
        // 必ず値が入っていることを保証する
        public ProductEntity(
            int productId, 
            string productName, 
            int price
            )
        {
            ProductId = productId;
            ProductName = productName;
            Price = price;
        }

        // プロパティ
        // 最初に設定した値を変更する必要がないのでgetプロパティのみ
        public int ProductId { get; }
        public string ProductName { get; }
        public int Price { get; }
    }
}
```

## SQLServerからCRUDするクラス

```c#
namespace Hoimi.SqlServer
{
    public static class ProductSqlServer
    {
        private static string _connectionString;

        // staticなコンストラクタ
        // 最初にこのクラスにアクセスしたとき、_connectionStringを使えるようになる
        static ProductSqlServer()
        {
            var builder = new SqlConnectionStringBuilder();
            builder.DataSource = @"localhost"; // SQLServerのサーバー名
            builder.InitialCatalog = "Hoimi"; // データベース名
            builder.IntegratedSecurity = true; // Windows認証で入る場合はtrue
            _connectionString = builder.ToString(); // 接続文字列
        }

        // DataTableを返すパターン
        // ① 接続先文字列の取得
        // ② 接続先文字列をSqlConnectionに設定
        // ③ SQLを発行するためにSqlDataAdapterを使う
        public static DataTable GetDataTable()
        {
            var sql = @"
SELECT * FROM Product
";
            DataTable dt = new DataTable();
            using (var connection = new SqlConnection(_connectionString))
            using (var adapter = new SqlDataAdapter(sql, connection))
            {
                connection.Open();
                adapter.Fill(dt);
            }

            return dt;
        }

        // DataReaderで取得するパターン
        // 一行ずつ読み込んで処理できる
        public static List<ProductEntity> GetDataReader()
        {
            var sql = @"
SELECT ProductId, ProductName, Price
FROM Product
";
            var result = new List<ProductEntity>();
            using (var connection = new SqlConnection(_connectionString))
            using (var command = new SqlCommand(sql, connection))
            {
                connection.Open();
                using (var reader = command.ExecuteReader())
                {
                    while (reader.Read())
                    {
                        // var ProductId = Convert.ToInt32(reader["ProductId"]);
                        // 文字列で項目名を指定する
                        result.Add(new ProductEntity(
                            Convert.ToInt32(reader["ProductId"]),
                            Convert.ToString(reader["ProductName"]),
                            Convert.ToInt32(reader["Price"])));
                    }
                }
            }

            return result;
        }

        public static void Insert(ProductEntity entity)
        {
            var sql = @"
INSERT INTO Product (ProductId, ProductName, Price)
VALUES (@ProductId, @ProductName, @Price)
";
            using (var connection = new SqlConnection(_connectionString))
            using (var command = new SqlCommand(sql, connection))
            {
                connection.Open();
                command.Parameters.AddWithValue("@ProductId", entity.ProductId);
                command.Parameters.AddWithValue("@ProductName", entity.ProductName);
                command.Parameters.AddWithValue("@Price", entity.Price);
                command.ExecuteNonQuery(); // SQLを実行
            }
        }

        public static void Update(ProductEntity entity)
        {
            var sql = @"
UPDATE Product
SET ProductName = @ProductName, Price = @Price
WHERE ProductId = @ProductId
";
            using (var connection = new SqlConnection(_connectionString))
            using (var command = new SqlCommand(sql, connection))
            {
                connection.Open();
                command.Parameters.AddWithValue("@ProductId", entity.ProductId);
                command.Parameters.AddWithValue("@ProductName", entity.ProductName);
                command.Parameters.AddWithValue("@Price", entity.Price);
                var updateCount = command.ExecuteNonQuery(); // SQLを実行
                // UpdateがなかったらInsert
                if (updateCount < 1)
                {
                    Insert(entity);    
                }
            }
        }

        public static void Delete(int productId)
        {
            var sql = @"
DELETE Product
WHERE ProductId = @ProductId
";
            using (var connection = new SqlConnection(_connectionString))
            using (var command = new SqlCommand(sql, connection))
            {
                connection.Open();
                command.Parameters.AddWithValue("@ProductId", productId);
                command.ExecuteNonQuery(); // SQLを実行
            }
        }
    }
}
```

## Dapperでデータを取得する方法
# 用語

完全コンストラクタパターン
# 外部リンク

[SQL Server データ型のマッピング](https://learn.microsoft.com/ja-jp/dotnet/framework/data/adonet/sql-server-data-type-mappings)
