# parsers
Writting of a parser of raw file content (of text) in order to extract useful informations (tokens and values).
In fact, with the parser, we should be able to get characters' id,description, images, and their states as well as we want.

# The ANTLR4 Grammar

```antlr
grammar CharacterStateSpecies;

options {
    caseInsensitive=true;
}

parse
    :   ( character | state | species | autoDescription )* EOF
    ;

character
    :   'CHARACTER' characterName '{'
        'character_id' ':' characterId?
        (   'description' ':' description
        |   'images' ':' imageUrlList
        |   'organ' ':' organName
        |   'states' ':' statesList
        )*
    '}'
    ;

state
    :   'STATE' stateName '{'
        'state_id' ':' stateId?
        (   'description' ':' description
        |   'images' ':' imageUrlList
        |   'character' ':' (characterName | characterId)?
        )*
    '}'
    ;

species
    :   'SPECIES' speciesName '{'
        'species_id' ':' speciesId?
        (   'details' ':' details
        |   'images' ':' imageUrlList
        )*
    '}'
    ;

autoDescription
    :   'AUTO_DESCRIPTION' speciesName '{' 'characterStates' ':' characterStateValueList '}'
    ;

characterId
    :   ( '[' 'CID' ']' | 'CID' ) INT
    ;

stateId
    :   ( '[' 'SID' ']' | 'SID' ) INT
    ;

speciesId
    :   ( '[' 'SPID' ']' | 'SPID' ) INT
    ;

characterName
    :   Identifier
    ;

organName
    :   Identifier
    ;

stateName
    :   Identifier
    ;

speciesName
    :   Identifier
    ;

statesList
    :   (stateName | stateId) (',' (stateName | stateId))*
    ;

characterStateValueList
    :   (characterStateValue ',')* characterStateValue
    ;

characterStateValue
    :   (characterId | characterName) ':' stateValue
    ;

stateValue
    :   nominalState ':' stateStatus
    |   numericValue
    |   alphanumericValue
    |   intervalValue
    ;

nominalState
    :   stateName | stateId
    ;

stateStatus
    :   'absent' | 'present' | 'inconnu' | 'unknown' | 'inapplicable'
    |   'présent' | '#' | '+' | '?' | '!' | '-' | '@'
    ;

numericValue
    :   '-'? (INT | FLOAT | MYREAL)
    ;

intervalValue
    :   interval unit?
    ;

MYREAL
    :   DecimalPart ('E' | 'e')? ExponentPart?
    |   DecimalPart ExponentPart
    ;

DecimalPart
    :   Digit* '.' Digit+
    |   Digit+
    ;

ExponentPart
    :   Sign? Digit+
    ;

Sign
    :   '+' | '-'
    ;

Digit
    :   [0-9]
    ;

alphanumericValue
    :   numericValue unit?
    ;

unit
    :   UnitChar+
    ;

description
    :   quotedText
    |   unquotedText
    ;

details
    :   quotedText
    |   unquotedText
    ;

quotedText
    :   '"' (~'"' | '\\"')* '"'
    ;

unquotedText
    :   TEXT
    ;

TEXT
    :   (~[\u0000-\u001F])+  // Allows all printable ASCII characters excluding C0 controls
    ;

imageUrlList
    :   url (',' url)*
    ;

url
    :   ('http://' | 'https://') restOfUrl
    |   filename
    |   filepath
    ;

restOfUrl
    :   (~(' ' | '\t' | '\r' | '\n'))+  // Exclude whitespace from URLs
    ;

filename
    :   Letter (Letter | Digit | '-')* '.' EXTENSION
    ;

filepath
    :   filename ('/' filename)*
    ;

EXTENSION
    :   '.' Letter (Letter | Digit | '-')*
    ;

Letter
    :   [a-zA-Z]
    ;

Identifier
    :   Letter (Letter | Digit | '_')*
    ;

WS
    :   [ \t\r\n]+ -> skip  // Ignore spaces, tabs, and newlines
    ;

UnitChar
    :   [a-zA-Z0-9]  // Allow letters and digits in a unit
    ;

interval
    :   '[' numericValue '..' numericValue ']'  // Closed interval
    |   '(' numericValue '..' numericValue ')'  // Open interval
    |   '[' numericValue '..' numericValue ')'  // Half-open interval (left-closed)
    |   '(' numericValue '..' numericValue ']'  // Half-open interval (right-closed)
    |   ']' numericValue '..' numericValue ']'  // Uncommon format, but included for completeness
    |   ']' numericValue '..' numericValue '['  // This might be a typo in the original request, as it's not a standard notation
    |   '[' numericValue '..' numericValue '['  // Another unusual case, but included
    |   '[' numericValue ';' numericValue ']'  // Closed interval
    |   '(' numericValue ';' numericValue ')'  // Open interval
    |   '[' numericValue ';' numericValue ')'  // Half-open interval (left-closed)
    |   '(' numericValue ';' numericValue ']'  // Half-open interval (right-closed)
    |   ']' numericValue ';' numericValue ']'  // Uncommon format, but included for completeness
    |   ']' numericValue ';' numericValue '['  // This might be a typo in the original request, as it's not a standard notation
    |   '[' numericValue ';' numericValue '['  // Another unusual case, but included
    |   '[' numericValue ',' numericValue ']'  // Closed interval
    |   '(' numericValue ',' numericValue ')'  // Open interval
    |   '[' numericValue ',' numericValue ')'  // Half-open interval (left-closed)
    |   '(' numericValue ',' numericValue ']'  // Half-open interval (right-closed)
    |   ']' numericValue ',' numericValue ']'  // Uncommon format, but included for completeness
    |   ']' numericValue ',' numericValue '['  // This might be a typo in the original request, as it's not a standard notation
    |   '[' numericValue ',' numericValue '['  // Another unusual case, but included
    |   numericValue '-' numericValue            // Range notation
    |   numericValue '..' numericValue            // Alternative range notation, assuming comma is used in some contexts for ranges
    |   '<' numericValue                         // Less than
    |   '>' numericValue                         // Greater than
    |   '<=' numericValue                        // Less than or equal to
    |   '=<' numericValue                        // This is usually represented as '<=', redundant unless specific context
    |   '>=' numericValue                        // Greater than or equal to
    |   '=>' numericValue                        // This seems like a custom operator, ensure it's intended
    |   numericValue '<'                         // Inverted order, might need clarification
    |   numericValue '>'                         // Inverted order
    |   numericValue '<='                        // Inverted order
    |   numericValue '=<'                       // Inverted order, redundant
    |   numericValue '>='                        // Inverted order
    |   numericValue '=>'                        // Inverted order
    ;
```
# The BNF syntax

```bnf

```
# A Concrete file content

```raw
CHARACTER "Couleur des fleurs"
{
    id: 1
    type: "NOMINAL"
    description: "Couleur des fleurs de la plante"
    is_multistate: "no"
    states_list: "blanc", "rouge", "jaune", "bleu", "rose"
}

STATE "blanc" { id: 1 }
STATE "rouge" { id: 2 }
STATE "jaune" { id: 3 }
STATE "bleu" { id: 4 }
STATE "rose" { id: 5 }

CHARACTER "Type de feuillage"
{
    id: 2
    type: "NOMINAL"
    states_list: "persistant", "caduc", "semi-persistant"
}

STATE "persistant" { id: 1 }
STATE "caduc" { id: 2 }
STATE "semi-persistant" { id: 3 }

CHARACTER "Forme des feuilles"
{
    id: 3
    type: "NOMINAL"
    description: "Forme générale des feuilles de la plante"
    states_list: "ovale", "lancéolée", "linéaire", "cordée", "réniforme", "hastée"
    images_list: "https://example.com/leaf-shapes.jpg"
}

STATE "ovale" { id: 1 }
STATE "lancéolée" { id: 2 }
STATE "linéaire" { id: 3 }
STATE "cordée" { id: 4 }
STATE "réniforme" { id: 5 }
STATE "hastée" { id: 6 }


CHARACTER "Présence d'épines"
{
    id: 4
    type: "NOMINAL"
    is_multistate: "no"
    states_list: "présentes", "absentes"
}

STATE "présentes" { id: 1 }
STATE "absentes" { id: 2 }

CHARACTER "Hauteur de la plante (cm)"
{
    id: 5
    type: "NUMERIC"
    description: "Hauteur maximale de la plante en centimètres"
    states_list: "[0..25]", "(25..50]", "(50..100]", "(100..200]", "(200..+Inf]"
}

STATE "[0..25]" { id: 1 }
STATE "(25..50]" { id: 2 }
STATE "(50..100]" { id: 3 }
STATE "(100..200]" { id: 4 }
STATE "(200..+Inf]" { id: 5 }


CHARACTER "Diamètre du tronc (cm)"
{
    id: 6
    type: "NUMERIC"
    states_list: "[0..10]", "(10..30]", "(30..+Inf]"
}

STATE "[0..10]" { id: 1 }
STATE "(10..30]" { id: 2 }
STATE "(30..+Inf]" { id: 3 }


CHARACTER "Longueur des feuilles (cm)"
{
    id: 7
    type: "NUMERIC"
    description: "Longueur des feuilles en centimètres"
    states_list: "[0..2]", "(2..5]", "(5..10]", "(10..20]", "(20..50]", "(50..+Inf]"
}

STATE "[0..2]" { id: 1 }
STATE "(2..5]" { id: 2 }
STATE "(5..10]" { id: 3 }
STATE "(10..20]" { id: 4 }
STATE "(20..50]" { id: 5 }
STATE "(50..+Inf]" { id: 6 }


CHARACTER "Présence de fleurs"
{
    id: 8
    type: "NUMERIC"
    is_multistate: "no"
    states_list: "0", "1"
}

STATE "0" { id: 1 }
STATE "1" { id: 2 }


CHARACTER "Température annuelle (°C)"
{
    id: 9
    type: "NUMERIC"
    states_list: "[-10..0]", "(0..10]", "[10..20]", "(20..30)", "30..-10"
}

STATE "[-10..0]" { id: 1 }
STATE "(0..10]" { id: 2 }
STATE "[10..20]" { id: 3 }
STATE "(20..30)" { id: 4 }

CHARACTER "Poids (kg)"
{
    id: 10
    type: "NUMERIC"
    states_list: "[0,5]", "(5;10]", "[10;20]", "(20,30)", ">=30"
}

STATE "[0,5]" { id: 1 }
STATE "(5;10]" { id: 2 }
STATE "[10;20]" { id: 3 }
STATE "(20,30)" { id: 4 }
STATE ">=30" { id: 5 }

CHARACTER "Âge (années)"
{
    id: 11
    type: "NUMERIC"
    states_list: "<5", ">=5", "<=10", ">10"
}

STATE "<5" { id: 1 }
STATE ">=5" { id: 2 }
STATE "<=10" { id: 3 }
STATE ">10" { id: 4 }

```
