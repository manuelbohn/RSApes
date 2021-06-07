# RSApes model

## Model basics

We have the following information to work with

* The context - psotive or negative
* The relationship between the communicators - is the dominant individual doing the signalling or the subordinate
* The gesture they used (SG or BG)
* The facial expression (bared teeth, hoot or neutral)

We use this to predict

* The reaction (affiliate or avoid)

## Model structure

We want to model the probability of the reaction given some utterance (gesture and facial expression) and the context. The context includes the social realtionship between individuals and immediate context of the utterance.

Context and social realtionship between individuals affect the prior over reactions. A negative context makes an avoid reaction more likely. The same holds for the dominance relation. Dominant individuals are more likely to show a avoid reaction compared to subordinates.

The gesture and the facial expression feed into the likelihood. They are produced by the communicator to express an intention (to affiliate or to avoid). However, the realtion between the intention and the gesture / facial expression are not deterministic, they are probabilistic.

## Model components

We have reactions, gestures and faces

```js
var all_reaction = [
    { type: "affiliate" },
    { type: "avoid" }
]

var gestures = ["stretched", "bent"]

var face = ["neutral", "hoot", "teeth"]
```

There are two possible lexica for gestures.

```js
var gesLexicon1 = function(utterance, reaction){
  utterance.gesture == "stretched" ? reaction.type == "affiliate" :
  utterance.gesture == "bent" ? reaction.type == "avoid" :
  true
  }

  var gesLexicon2 = function(utterance, reaction){
  utterance.gesture == "stretched" ? reaction.type == "avoid" :
  utterance.gesture == "bent" ? reaction.type == "affiliate" :
  true
  }
```

and I guess two differnt lexica for facial expressions, with neutral facial expressions being non informative with respect to the underlying intention.

```js
var faceLexicon1 = function(utterance, reaction){
  utterance.face == "hoot" ? reaction.type == "affiliate" :
  utterance.face == "teeth" ? reaction.type == "avoid" ?
  utterance.face == "neutral" ? flip() ? reaction.type == "affiliate" : reaction.type == "avoid" :
  true
  }

var faceLexicon1 = function(utterance, reaction){
  utterance.face == "hoot" ? reaction.type == "avoid" :
  utterance.face == "teeth" ? reaction.type == "affiliate" ?
  utterance.face == "neutral" ? flip() ? reaction.type == "affiliate" : reaction.type == "avoid" :
  true
  }
```

But then again, maybe we don't need any lexica since the "meaning" of the different gestures or facial expressions is assumed to be known to the listnener, nothing that needs to be inferred. So maybe we just need a meaning function that links gestures and facial expressions to reactions with some uncertainty? This meaning function could also change with the context, like you describe in section 3.1 of your problang paper.

```js
var gesMeaning = function(utterance){
  utterance.gesture == "stretched" ? flip(stretchMean) ? reaction.type == "avoid" : reaction.type == "affiliate" :
  utterance.gesture == "bent" ? flip(bentMean) ? reaction.type == "avoid" : reaction.type == "affiliate" :
  true
}
```

and the same for facial expressions?

```js
var faceMeaning = function(utterance){
  utterance.face == "hoot" ? flip(hootMean) ? reaction.type == "avoid" : reaction.type == "affiliate" :
  utterance.face == "teeth" ? flip(teethMean) ? reaction.type == "avoid" : reaction.type == "affiliate" :
  utterance.face == "neutral" ? flip() ? reaction.type == "affiliate" : reaction.type == "avoid" :
  true
}
```

What to do with the prior? I guess we could simply say that the context and the relationship are two indpendent components. Both could be a biased coin flip in the direction specified above. Their exact weight could be inferred or fixed.

```js
var contextPrior = 0.75

var relationPrior = 0.75
```

How to put that all together in a literal listener? The listener takes in an utterance, which is a combination of a gesture and a facial expression. And then it takes in the context and the relationship. Question is how the context and the relationship priors relate to one another. We could simply multiply them? I guess it would be nice to spell this out eventually.

```js
var all_reactions = [
  { type: "affiliate" },
  { type: "avoid" }
]

var literalListener = function(utterance, context, relationship){
  Infer({method: "enumerate", model: function(){
    // building up the prior
    var priorProbContext = (context == "positive") ? contextPrior : 1-contextPrior
    var priorProbRelation = (relationship == "todom") ? relationPrior : 1-relationPrior
    var priorProbs = normalize([
      priorProbContext * priorProbRelation,
      (1 - priorProbContext) * (1 - priorProbRelation)
    ])

    // taking in the different parts of the utterance
    var reaction = sample( Categorical({vs: all_reactions, ps: priorProbs}))

    var gesTruthValue = gesMeaning(utterance)
    var faceTruthValue = faceMeaning(utterance)

    // Do some conditioning I guess
    condition(gesTruthValue)
    condition(faceTruthValue)

    // return the inferred reaction
    return reaction.type
  }})
}
```

All of this doesn't seem right ...
