# mnnllm (MNN-LLM Flutter Plugin)

A Flutter Plugin for Android and IOS based on MNN-LLM.

|             | Android |    iOS   |
|-------------|---------|----------------|
| **Support** | SDK 26+ |  *Not Supported* |

## Getting Started

Add mnnllm as a dependency in your pubspec.yaml file.

```yarml
dependencies:
  mnnllm: ^{{latest version}}
```

Replace {{latest version}} with the version number in badge above.

## Example

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:mnnllm/mnnllm.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  TextEditingController _controller = TextEditingController();
  String performanceText = "";
  String downloadingState = "";
  String generateText = "";
  String inputText = "Hello?";
  String modelID = "taobao-mnn/Qwen3-0.6B-MNN";
  bool think = true;
  bool allowGen = false;
  final _mnnllmPlugin = MnnLLM();

  @override
  void initState() {
    super.initState();
    _controller.text = inputText;
  }

  // Initialize Model for chat
  void initModel() async {
    String? value = await _mnnllmPlugin.download(modelID);
    if (value != null) {
      setState(() {
        downloadingState = value;
      });
    }
    String? configPath = await _mnnllmPlugin.getConfigPath(modelID);
    if (configPath == null) {
      print("Null Error");
      return;
    }
    int? val = await _mnnllmPlugin.initModel(
        modelID,
        _mnnllmPlugin.getModelName(modelID),
        configPath,
        _mnnllmPlugin.getSessionId(null),
        keepHistory: false
    );
    if (val == 0) {
      print("load success");
    }

    while (true) {
      await Future.delayed(Duration(seconds: 1));
      int? code = await _mnnllmPlugin.modelInitState();
      if (code == 0) {
        break;
      }
    }

    // Set Max Tokens of Generation
    await _mnnllmPlugin.setMaxTokens(300);
    // Set Non-Think mode
    await _mnnllmPlugin.setThinking(modelID, false);
    think = false;
    allowGen = true;
  }

  // Download Model from HuggingFace Or ModelScope
  void downloadLLM() async {
    String? provider = await _mnnllmPlugin.getDownloaderProvider();
    print("Provider : $provider");
    String? value = await _mnnllmPlugin.download(modelID);
    if (value != null) {
      setState(() {
        downloadingState = value;
        print(downloadingState);
      });
    }
  }

  // Stop Generating Tokens
  void stopGen() async {
    int? value = await _mnnllmPlugin.stopGenerate();
    if (value == 0) {
      print("Stop Success");
    }
  }

  // Switch between Think or Non-Think
  void thinkSwitch() async {
    int? value = await _mnnllmPlugin.setThinking(
        modelID, !think);
    if (value == 0) {
      think = !think;
      print("Think Switch Success");
    }
  }

  // Generate Tokens in stream mode
  void streamGenerate() async {
    _mnnllmPlugin.generateStreamAnswer(inputText,
            (data) { setState(() { generateText = data.toString(); }); },
            (err) { print(err.toString()); });

    while (true) {
      await Future.delayed(const Duration(seconds: 1));
      int? finished = await _mnnllmPlugin.isGenerateFinished();
      if (finished != null && finished == 0) {
        String? per = await _mnnllmPlugin.getPerformance();
        setState(() {
          performanceText = per as String;
        });
        break;
      }
    }
  }

  // Generate Tokens in a once
  Future<void> finalGenerate() async {
    String? ans = await _mnnllmPlugin.generateFinalAnswer(inputText);
    String? per = await _mnnllmPlugin.getPerformance();
    if (ans != null) {
      setState(() {
        generateText = ans;
        performanceText = per as String;
      });
      return;
    }
    print("Error Generate");
  }

  @override
  void dispose() {
    // TODO: implement dispose
    _mnnllmPlugin.destroyLLM();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('MNN-LLM APP'),
        ),
        body: ListView(
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                OutlinedButton(onPressed: () { downloadLLM(); }, child: Text("Download LLM")),
                OutlinedButton(onPressed: () { initModel(); }, child: Text("Initial Model"))
              ],
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                OutlinedButton(onPressed: () { thinkSwitch(); }, child: Text("Think Switch")),
                OutlinedButton(onPressed: () { stopGen(); }, child: Text("Stop Generating"))
              ],
            ),
            const Text(""),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                SizedBox(width: 280, child: TextField(onChanged: (val) { inputText = val; }, controller: _controller,)),
              ],
            ),
            const Text(""),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                OutlinedButton(onPressed: () { if(allowGen) { streamGenerate();} }, child: Text("Stream Generate")),
                OutlinedButton(onPressed: () async { if(allowGen) { await finalGenerate();} }, child: Text("Final Generate"))
              ],
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.start,
              children: [
                Text("Downloading State : "),
                SizedBox(
                  width: 200,
                  child: Text(downloadingState, overflow: TextOverflow.ellipsis,),
                )
              ],
            ),
            Divider(color: Colors.black87),
            Row(mainAxisAlignment: MainAxisAlignment.start,
              children: [
                Text("Performance:"),
                SizedBox(width: 260, child: Text(performanceText, maxLines: 200, softWrap: true,),)
              ],),
            Divider(color: Colors.black87,),
            Row(mainAxisAlignment: MainAxisAlignment.start,
            children: [
              Text("Generating:"),
              SizedBox(width: 260, child: Text(generateText, maxLines: 200, softWrap: true,),)
            ],),
            // SizedBox(width: 300, child: const StreamText(),)
          ],
        )
      ),
    );
  }
}
```

## Runtime Screenshot

![The example app running on Kirin 960](https://github.com/liuzhi19121999/mnnllm/blob/main/assets/kirin960.gif?raw=true) ![The example app running on Exynos 880](https://github.com/liuzhi19121999/mnnllm/blob/main/assets/exynos880.gif?raw=true)

## Warnings

The development work for the iOS version is still proceeding at an accelerated pace.

## RoadMap

- Support for iOS platform
- Support for text-to-image generation models
- Support for multimodal large models

## Contact Me

Liu `liuzhi1999@foxmail.com`
