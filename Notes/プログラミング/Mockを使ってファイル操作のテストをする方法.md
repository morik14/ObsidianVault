---
tags:
  - プログラミング
  - CSharp
---
# Mockを使わないサンプルコード

## テストされる側

```c#
namespace TDD.WinForm.Objects
{
    public class TextFile
    {
        public bool NewLineExists()
        {
            var fileString = File.ReadAllText(@"C:\work\Test.txt");
            return fileString.Contains(System.Environment.NewLine);
        }
    }
}
```

## テストする側

```c#
namespace TDDTest.Tests
{
    [TestClass]
    public class UnitTest2
    {
        [TestMethod]
        public void 改行コードの有無をチェックする()
        {
            var textFile = new TextFile();
            Assert.AreEqual(true, textFile.NewLineExists());
        }
    }
}
```

## 問題点

正常系と異常系を同時にテストできない

# Mockを使うサンプルコード

## テストされる側
### インターフェース

```c#
namespace TDD.WinForm.Objects
{
    public interface ITextFile
    {
        string GetData();
    }
}
```

### インターフェースを実装するクラス

```c#
namespace TDD.WinForm.Objects
{
    public class TextFileAccess : ITextFile
    {
        public string GetData()
        {
            return File.ReadAllText(@"C:\work\Test.txt");
        }
    }
}
```

### テストが必要な部分

```c#
namespace TDD.WinForm.Objects
{
    public class TextFile
    {
        private ITextFile _textFile;

        // TextFileを指定するコンストラクタ
        public TextFile(ITextFile textFile)
        {
            _textFile = textFile;
        }

        public bool NewLineExists()
        {
            var fileString = _textFile.GetData();
            return fileString.Contains(System.Environment.NewLine);
        }
    }
}
```

## テストする側

### Mockを作る

```c#
namespace TDDTest.Tests
{
    // Testプロジェクトからしか参照しないので
    // 明示的にinternalをつける
    internal class TextFileMock : ITextFile
    {
        // ValueプロパティでMockが返却する値を操作できるように
        internal string Value { get; set; }

        public string GetData()
        {
            return Value;
        }
    }
}
```


### Mockを使ってテスト

```c#
namespace TDDTest.Tests
{
    [TestClass]
    public class UnitTest2
    {
        [TestMethod]
        public void 改行コードの有無をチェックする正常系()
        {
            var textFileMock = new TextFileMock();
            textFileMock.Value = "AAA" + System.Environment.NewLine;
            var textFile = new TextFile(textFileMock);
            Assert.AreEqual(true, textFile.NewLineExists());
        }

        [TestMethod]
        public void 改行コードの有無をチェックする異常系()
        {
            var textFileMock = new TextFileMock();
            textFileMock.Value = "BBB";
            var textFile = new TextFile(textFileMock);
            Assert.AreEqual(false, textFile.NewLineExists());
        }
    }
}
```

### ポイント

外部と接続する部分をMock化している
返却する値を外部から変更して、正常系と異常系のテストができる
Mockは実際にファイルにアクセスしていない