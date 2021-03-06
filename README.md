# Food Explanation Ontology
Ontology for generating Food Related Explanation

## Setup

**1. Prepare the virtual environment**

```
python -m pip install virtualenv
python -m virtualenv env --python=python3
source env/bin/activate
python -m pip install -r requirements.txt
```

**2. Run the code**

Run the SPARQL queries on the inferred axioms from ```feo.ttl```.

Protege Directions:
- Use the *Export inferred axioms* option in the *File*. 
- On the window that comes up, select all boxes, except for the last (*Disjoint Classes*)." 

Alternatively, one may also run the reasoner separately, save the inferred values, and import to Blazegraph, or another similar knowledge graph store.

## Competency Questions

1. **Contextual** - "Why should I eat Cauliflower Potato Curry?"

```
PREFIX feo: <http://purl.org/heals/food-explanation-ontology/>
PREFIX eo: <http://purl.org/heals/eo#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?characteristic ?classes
WHERE{
  ?WhyEatCauliflowerPotatoCurry feo:hasParameter ?parameter .
  ?parameter feo:hasCharacteristic ?characteristic .
  ?characteristic feo:isInternal False .
  ?systemChar a feo:SystemCharacteristic .
  ?userChar a feo:UserCharacteristic .
  filter ( ?characteristic = ?systemChar || ?characteristic = ?userChar ) .
  ?characteristic a ?classes .
  ?classes rdfs:subClassOf feo:Characteristic .
  Filter Not Exists{?classes rdfs:subClassOf <https://purl.org/heals/eo#knowledge> }.
}
```

2. **Counterfactual** - "Why should I eat Butternut Squash Soup over a Broccoli Cheddar Soup?"

```
PREFIX food: <http://purl.org/heals/food/>
PREFIX eo: <http://purl.org/heals/eo#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

Select DISTINCT ?factType ?factA ?foilType ?foilB
Where{
  ?question feo:hasPrimaryParameter ?parameterA .
  ?question feo:hasSecondaryParameter ?parameterB .

  ?parameterA feo:hasCharacteristic ?factA .
  ?factA a <https://purl.org/heals/eo#Fact>.
  ?factA a ?factType .
  ?factType (rdfs:subClassOf+) feo:Characteristic .
  Filter Not Exists{?factType rdfs:subClassOf <https://purl.org/heals/eo#knowledge> }.
  Filter Not Exists{?s rdfs:subClassOf ?factType}.
  
  ?parameterB feo:hasCharacteristic ?foilB .
  ?foilB a <https://purl.org/heals/eo#Foil> .
  ?foilB a ?foilType.
  ?foilType (rdfs:subClassOf+) feo:Characteristic .
  Filter Not Exists{?foilType rdfs:subClassOf <https://purl.org/heals/eo#knowledge> }.
  Filter Not Exists{?t rdfs:subClassOf ?foilType}.

}
```

3. **Contrastive** - "What if I was pregnant?"

```

PREFIX feo: <http://purl.org/heals/food-explanation-ontology/>
PREFIX food: <http://purl.org/heals/food/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 

SELECT Distinct ?property ?ingredient ?outputs
WHERE{
  feo:WhatIfIWasPregnant  feo:hasParameter ?parameter .
  ?parameter ?property  ?ingredient .
  ?property rdfs:subPropertyOf feo:isCharacteristicOf.
  ?ingredient a food:Food .
  OPTIONAL { ?ingredient feo:isIngredientOf ?outputs.}
}

```

4. **Statistical** - "Why should I follow a low calorie diet?"

```
PREFIX feo: <http://purl.org/heals/food-explanation-ontology/>
PREFIX eo: <http://purl.org/heals/eo#>

SELECT DISTINCT ?goal ((Count( distinct ?accomplishedUsers))/(Count( distinct ?allOtherUsers)) as ?percentAccomplished)

WHERE{
  
  ?WhyFollowLowCalorieDiet feo:hasParameter ?diet.
  ?WhyFollowLowCalorieDiet feo:askedBy ?mainUser.
  ?mainUser feo:hasGoal ?goal.
  ?allOtherUsers feo:hasDiet ?diet .
  FILTER ( ?allOtherUsers != ?mainUser ).
  ?accomplishedUsers feo:hasDiet ?diet .
  ?accomplishedUsers feo:accomplishedGoal ?goal.
}
GROUP BY ?goal
Having ((Count( distinct ?accomplishedUsers))/(Count( distinct ?allOtherUsers)) >= .5)
```


