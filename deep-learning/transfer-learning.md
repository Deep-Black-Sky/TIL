# 転移学習
Kerasを用いた転移学習

## TL;TR
大規模なモデル(VGG16, ResNet)を利用して、低コストで高パフォーマンスを期待できる。

## 転移学習とは
> 転移学習とは、端的に言えばある領域で学習させたモデルを、別の領域に適応させる技術です。
> 具体的には、広くデータが手に入る領域で学習させたモデルを少ないデータしかない領域に適応させたり、シミュレーター環境で学習させたモデルを現実に適応させたりする技術です。
> これにより、少ないデータしかない領域でのモデル構築や、ボンネットに立つという危険を侵さずにモデルを構築することができるというわけです。
>
> [転移学習:機械学習の次のフロンティアへの招待](https://qiita.com/icoxfog417/items/48cbf087dd22f1f8c6f4)

大規模なデータで学習された、汎化性の優れたモデルを特徴量抽出機として利用する。
これにより少量のデータに於いても十分な分散を持つ特徴量が得られ、ごく小さなNeuralNetworkでも十分なパフォーマンスが得られると期待される。

## Kerasでの実装
[code](https://github.com/Deep-Black-Sky/transfer-training)

```python
# Model.py

def construct_model(num_classes):
    # load ResNet model.
    input = Input(shape=(224, 224, 3))
    # include_top means remove the dense layers.
    resnet = ResNet50(include_top=False,
                     weights='imagenet',
                     input_tensor=input)

    # construt dense layers
    top_model = Sequential()
    top_model.add(Flatten(input_shape=resnet.output_shape[1:]))
    top_model.add(Dense(256))
    top_model.add(Activation('relu'))
    top_model.add(Dropout(0.5))
    top_model.add(Dense(num_classes))
    top_model.add(Activation('softmax'))

    # stack dense layers over ResNet model.
    model = Model(input=resnet.input, output=top_model(resnet.output))

    # fix wights of ResNet
    for layer in model.layers[:len(model.layers) - 1]:
        layer.trainable = False

    model.compile(loss='categorical_crossentropy',
                  optimizer=Adam(lr=1e-3),
                  metrics=['accuracy'])

    return model
```

- Keras APIでResNetを読み込み、その出力層を削除する(include_top=False)。
- 新たにDense Layerを作成。ResNetモデルとそれらで新たにモデルを作成する。
- ResNetの重みを固定(学習不可能)する(layer.trainable = False)。

## What to do next
- Kerasで用意されているモデル以外での転移学習。WaveNet Decoderを使いたい。
