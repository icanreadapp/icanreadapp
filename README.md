## Hi there 👋
import 'package:flutter/material.dart';
import 'package:audioplayers/audioplayers.dart';
import 'dart:async';
import 'dart:math';

void main() {
  runApp(ICanReadApp());
}

class ICanReadApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'I Can Read',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  final List<String> levels = List.generate(50, (index) => 'Level ${index + 1}');

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('I Can Read'),
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: Image.asset('assets/markus.png', height: 100),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: levels.length,
              itemBuilder: (context, index) {
                return Card(
                  margin: EdgeInsets.all(10),
                  child: ListTile(
                    title: Text(levels[index]),
                    trailing: Icon(Icons.arrow_forward),
                    onTap: () {
                      Navigator.push(
                        context,
                        MaterialPageRoute(
                          builder: (context) => LessonScreen(level: levels[index], levelIndex: index),
                        ),
                      );
                    },
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

class LessonScreen extends StatefulWidget {
  final String level;
  final int levelIndex;

  LessonScreen({required this.level, required this.levelIndex});

  @override
  _LessonScreenState createState() => _LessonScreenState();
}

class _LessonScreenState extends State<LessonScreen> {
  final AudioPlayer _audioPlayer = AudioPlayer();
  bool _rewardVisible = false;
  int _correctAnswers = 0;
  int _totalQuestions = 10;

  late final String levelPhrase;
  late final List<String> levelSounds;
  late final List<String> levelWords;
  late final Map<String, String> wordImages;
  late final List<String> sentenceBuilder;

  List<String> currentSentenceParts = [];
  String? currentTargetSentence;

  @override
  void initState() {
    super.initState();

    if (widget.levelIndex == 0) {
      levelPhrase = 'I want';
      levelSounds = ['t', 'short a', 'n', 'w', 'long i'];
      levelWords = ['cat', 'a', 'bat', 'hat'];
      wordImages = {
        'cat': 'assets/images/cat.png',
        'bat': 'assets/images/bat.png',
        'hat': 'assets/images/hat.png',
      };
      sentenceBuilder = [];
    } else if (widget.levelIndex == 1) {
      levelPhrase = 'I want';
      levelSounds = ['c', 'b', 'h', 't', 'long a'];
      levelWords = ['a', 'hat', 'cat', 'bat'];
      wordImages = {
        'cat': 'assets/images/cat.png',
        'bat': 'assets/images/bat.png',
        'hat': 'assets/images/hat.png',
      };
      sentenceBuilder = [
        'I want a cat',
        'I want a bat',
        'I want a hat',
      ];
    } else if (widget.levelIndex == 2) {
      levelPhrase = 'I am';
      levelSounds = ['long a', 'm', 'l', 'd', 'c'];
      levelWords = ['cold', 'mad', 'calm'];
      wordImages = {
        'cold': 'assets/images/cold.png',
        'mad': 'assets/images/mad.png',
        'calm': 'assets/images/calm.png',
      };
      sentenceBuilder = [
        'I am cold',
        'I am calm',
        'I am mad',
      ];
    } else if (widget.levelIndex == 3) {
      levelPhrase = 'I am';
      levelSounds = ['p', 's', 't', 'short o', 'vowel y'];
      levelWords = ['happy', 'sad', 'hot'];
      wordImages = {
        'happy': 'assets/images/happy.png',
        'sad': 'assets/images/sad.png',
        'hot': 'assets/images/hot.png',
      };
      sentenceBuilder = [
        'I am happy',
        'I am sad',
        'I am hot',
      ];
    } else {
      levelPhrase = '';
      levelSounds = [];
      levelWords = [];
      wordImages = {};
      sentenceBuilder = [];
    }

    if (sentenceBuilder.isNotEmpty) {
      currentTargetSentence = (sentenceBuilder..shuffle()).first;
      currentSentenceParts = currentTargetSentence!.split(' ')
        ..shuffle();
    }
  }

  void _playPhrase() async {
    await _audioPlayer.play(AssetSource('audio/phrase_i_want.mp3'));
    setState(() {
      _rewardVisible = true;
    });
  }

  double get mastery => _correctAnswers / _totalQuestions;

  void _answerQuestion(bool correct) {
    if (correct) {
      setState(() {
        _correctAnswers++;
      });
    }
  }

  bool get canAdvance => mastery >= 0.9;

  void _checkSentence(List<String> parts) {
    if (currentTargetSentence == parts.join(' ')) {
      _answerQuestion(true);
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Correct sentence!')));
      setState(() {
        currentTargetSentence = (sentenceBuilder..shuffle()).first;
        currentSentenceParts = currentTargetSentence!.split(' ')
          ..shuffle();
      });
    } else {
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Try again.')));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.level),
      ),
      body: SingleChildScrollView(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Welcome to ${widget.level}!',
              textAlign: TextAlign.center,
              style: TextStyle(fontSize: 24),
            ),
            SizedBox(height: 20),
            if (levelPhrase.isNotEmpty)
              ElevatedButton(
                onPressed: _playPhrase,
                child: Text('Play Phrase: "$levelPhrase"'),
              ),
            if (levelSounds.isNotEmpty)
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: Text('Sounds: ${levelSounds.join(", ")}', style: TextStyle(fontSize: 18)),
              ),
            if (levelWords.isNotEmpty)
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: Text('Words: ${levelWords.join(", ")}', style: TextStyle(fontSize: 18)),
              ),
            Wrap(
              spacing: 10,
              children: levelWords.where((word) => wordImages.containsKey(word)).map((word) {
                return Column(
                  children: [
                    Image.asset(wordImages[word]!, height: 100),
                    Text(word, style: TextStyle(fontSize: 16)),
                  ],
                );
              }).toList(),
            ),
            if (sentenceBuilder.isNotEmpty && currentTargetSentence != null)
              Column(
                children: [
                  Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: Text('Tap to form: "$currentTargetSentence"', style: TextStyle(fontSize: 18)),
                  ),
                  Wrap(
                    spacing: 8.0,
                    runSpacing: 4.0,
                    children: currentSentenceParts.map((word) {
                      return ElevatedButton(
                        onPressed: () {
                          setState(() {
                            _selectedWords.add(word);
                            currentSentenceParts.remove(word);
                          });
                        },
                        child: Text(word, style: TextStyle(fontSize: 18)),
                      );
                    }).toList(),
                  ),
                  Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: Text('Your Sentence: ${_selectedWords.join(" ")}', style: TextStyle(fontSize: 18)),
                  ),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      ElevatedButton(
                        onPressed: () => _checkSentence(_selectedWords),
                        child: Text('Check Sentence'),
                      ),
                      SizedBox(width: 10),
                      ElevatedButton(
                        onPressed: () {
                          setState(() {
                            currentSentenceParts.addAll(_selectedWords);
                            _selectedWords.clear();
                            currentSentenceParts.shuffle();
                          });
                        },
                        child: Text('Reset'),
                      ),
                    ],
                  ),
                ],
              ),
            if (_rewardVisible)
              Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  children: [
                    Icon(Icons.star, color: Colors.amber, size: 64),
                    Text('Great job!', style: TextStyle(fontSize: 20))
                  ],
                ),
              ),
            Text('Progress: ${(_correctAnswers / _totalQuestions * 100).toStringAsFixed(0)}%', style: TextStyle(fontSize: 18)),
            if (canAdvance)
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: Text('You can advance to the next level!', style: TextStyle(fontSize: 18, color: Colors.green)),
              )
            else
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: Text('Reach 90% to unlock the next level.', style: TextStyle(fontSize: 18, color: Colors.red)),
              ),
            ElevatedButton(
              onPressed: () => _answerQuestion(true),
              child: Text('Simulate Correct Answer'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => MiniGameScreen(),
                  ),
                );
              },
              child: Text('Play Mini-Game'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => RunGameScreen(),
                  ),
                );
              },
              child: Text('Play Run Game'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => RacingGameScreen(),
                  ),
                );
              },
              child: Text('Play Racing Game'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => GestureMatchGameScreen(),
                  ),
                );
              },
              child: Text('Play Gesture Match Game'),
            ),
          ],
        ),
      ),
    );
  }

  List<String> _selectedWords = [];
}

// Other game classes remain unchanged...


<!--
**icanreadapp/icanreadapp** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
