## Flutter Demo

[Flutter Api Docs](https://flutter.cn/docs/development/ui/widgets)

### 1. 显示随机生成的单词

```dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
    const MyApp({Key? key}) : super(key: key);

    @override
    Widget build(BuildContext context) {
        return MaterialApp(	// 1
            title: 'Welcome to Flutter',	// 进程名称
            home: Scaffold(	// 2
                appBar: AppBar(	// 3
                    title: const Text('Welcome to Flutter'),
                ),
                body: Center(
                    child: RandomWords(),
                ),
            ),
        );
    }
}

// 有状态的组件，可以创建一个状态
class RandomWords extends StatefulWidget {
    @override
    _RandomWordsState createState() => _RandomWordsState();
}

// 状态
class _RandomWordsState extends State<RandomWords> {
    @override
    Widget build(BuildContext context) {
        var wordPair = WordPair.random();	// 随机生成两个单词的组合
        return Text(wordPair.asPascalCase);	// asPascalCase是让单词首字母大写
    }
}
```

1. **MaterialApp：**类似于网页中的`<html> </html>`，且它自带路由、主题色、主题等功能。

2. **Scaffold：**会填充可用空间，占据整个窗口或设备屏幕。Scaffold提供了大多数应用程序都应该具备的功能，例如顶部的`appBar`，底部的`bottomNavigationBar`，隐藏的侧边栏`drawer`等。

3. **AppBar：**应用栏

   

### 2. 列表

```dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
    const MyApp({Key? key}) : super(key: key);

    @override
    Widget build(BuildContext context) {
        return MaterialApp(title: 'Startup Name Generator', home: RandomWords());
    }
}

class RandomWords extends StatefulWidget {
    @override
    _RandomWordsState createState() => _RandomWordsState();
}

class _RandomWordsState extends State<RandomWords> {
    final _suggestions = <WordPair>[];
    final _biggerFont = const TextStyle(fontSize: 18.0);

    @override
    Widget build(BuildContext context) {
        return Scaffold(
            appBar: AppBar(
                title: const Text("Startup Name Generator"),
            ),
            body: buildSuggestions(),
        );
    }

  // 创建ListView,每个表项都会调用一次itemBuilder
    Widget buildSuggestions() {
        return ListView.builder(
            padding: const EdgeInsets.all(16.0),
            itemBuilder: (context, i) {
                if (i.isOdd) return const Divider();

                final index = i ~/ 2;
                if (index >= _suggestions.length) {
                    //generateWordPairs()是生成两音节单词，take(10)表示生成10个
                    _suggestions.addAll(generateWordPairs().take(10));	
                }
                return _buildRow(_suggestions[index]);
            });
    }

    // 表项
    Widget _buildRow(WordPair pair) {
        return ListTile(
            title: Text(
                pair.asPascalCase,
                style: _biggerFont,
            ),
        );
    }
}
```



### 3. 点击按钮跳转页面

```dart
class _RandomWordsState extends State<RandomWords> { 
    @override
    Widget build(BuildContext context) {
        return Scaffold(
            appBar: AppBar(
                title: const Text("Startup Name Generator"),
                actions: <Widget>[
                    IconButton(icon: const Icon(Icons.list), onPressed: _pushSaved),
                ],	// 给按钮添加点击跳转方法
                backgroundColor: Colors.white,
                foregroundColor: Colors.black,
            ),
            body: buildSuggestions(),
        );
    }


    // 处理点击事件的方法，导航中添加route，route中构建一个Scaffold
    void _pushSaved() {
        Navigator.of(context).push(
            MaterialPageRoute(
                builder: (BuildContext context) {
                    final Iterable<ListTile> tiles = _saved.map(
                        (WordPair pair) => ListTile(
                            title: Text(
                                pair.asPascalCase,
                                style: _biggerFont,
                            ),
                        ),
                    );
                    final List<Widget> divided = ListTile.divideTiles(
                        tiles: tiles,
                        context: context,
                    ).toList();

                    return Scaffold(
                        appBar: AppBar(
                            title: const Text("Saved Suggestions"),
                        ),
                        body: ListView(
                            children: divided,
                        ),
                    );
                },
            ),
        );
    }
}
```

