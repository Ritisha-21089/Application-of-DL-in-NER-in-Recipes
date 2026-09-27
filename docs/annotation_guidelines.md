**Definitions for each tag of an ingredient phrase**  
---

1) **Quantity:** The numerical value that indicates how much of an ingredient is required in a recipe. It typically appears at the beginning of an ingredient phrase.  
   Example:   
- “2 cups of flour” → Quantity: 2  
    
2) **Unit:** The measurement unit associated with the quantity, specifying how the ingredient is measured. It can be standard (e.g., grams, cups, tablespoons) or non-standard (e.g., a pinch, a handful).  
   Example: “2 cups of flour” → Unit: cups  
     
3) **Ingredient Name**: The primary name of the ingredient being used in the recipe. It is the core component of the ingredient phrase, excluding additional descriptors like form or state.  
   Example: “2 cups of flour” → Ingredient Name: flour  
     
4) **Form:** The specific physical or structural presentation of the ingredient, such as whole, chunks, slices, powder, juice, cubes, pieces, colour (describing the type), leaves, leaf, pods, long-grain, plain, concentrate or flat.   
   Example: “1 onion, finely chopped” → Form: finely chopped  
     
5) **State:** The condition of the ingredient or any treatment applied to the ingredient, indicating how it is.

   To remain consistent and avoid any confusion between state and form we have defined these rules:

- Any past participle present before the ingredient name as state.   
- Words like fat-free, gluten-free, day old, sweet, spicy, mixed, and flavoured have been considered as state.   
  Example: “½ cup of melted butter” → State: melted  
    
6) **Dry/Fresh:** A sub-category of state indicating whether an ingredient is in its dried form or fresh form. When the state of the ingredient is mentioned as dry, sun-dried, dried or fresh.  
   	  
   Example: “1 tsp dried basil” → Dry/Fresh: dried

   Example: “1 tbsp fresh parsley” → Dry/Fresh: fresh

   

7) **Size:** The measurements of the ingredient after application of action on it or the dimension and proportions pre-defined for the same.

		Example: “3 small zucchini, cut into 1/2-inch pieces” → Size: small, 1/2-inch

 **Extra Rules:**  
While annotating the dataset, we have decided to drop the segments written:

- Within brackets i.e. \-LRB- to \-RRB-   
- Phrase present after the preposition *“or”*

**Let's map the tags to letters for simplicity**

1. 'INGREDIENT NAME': N  
2.  'STATE': S  
3.  'QUANTITY': Q  
4.  'UNIT': U  
5.  'DRY/ FRESH': D  
6.  'SIZE': S  
7.  'FORM': F  
8.  'PREPROCESSING': P  
9.  ‘OTHERS’: O

1) 1 banana , peeled and cut half lengthwise and each half cut into 6 pieces  
   \[ Q N O P O P F O O O F P O Q F \]  
   *Words like lengthwise or diagonal are being considered as O. Pieces or any other dimension or shape of an ingredient after post-preprocessing are also tagged as form.*   
     
2) 2 chilies , red or green seeded and minced  
   \[ Q N O F O O O O O\]  
   *After ‘or’ everything is tagged as O and colours like red are getting the form tag.*   
     
3) 2 large cooking apples , peeled and cut into pieces  
   \[Q S O N O P O P O F\]  
   *Obvious words that don’t add meaning to the ingredient’s quality are given the O tag like cooking apples, or boiling potatoes.*  
     
4) 1 jar thick and chunky pasta sauce , 2-3/4 cups \-LRB- or spaghetti sauce \-RRB-  
   \[Q U S O F N N O Q U O O O O O\]  
 


5) 3 slices of day-old bread , preferably day-old French bread , broken into small pieces  
   \[Q U O S N O O S N N O P O S F\]

6) 1 lb beef tenderloin , sliced into 4 steaks  
   \[Q U N N O P O Q N\]  
   *Although words like pieces, strips and cubes are tagged as form but steaks are an ingredient itself and not a shape, hence given the tag name.*  
     
7) 1 lb roughy salmon fillet , cut into 4 pieces  
   \[Q U F N N O P O Q F\]  
     
8) 2 whole garlic cloves \-LRB- minced \-RRB-  
   \[Q U N F O O O\] 

*Whole represents unit, when a regular unit is not mentioned. Cloves usually represent forms.*

9) 10 \-15 cloves garlic  
   \[Q Q U N\]  
   *Here cloves represent unit and not form*  
     
10) 7 corn cakes , thawed if frozen \-LRB- arepas \-RRB-  
    \[Q N N O P O S O O O\]  
    *Here you are performing the action of thawed (preprocessing) if the ingredient is already in the frozen state.*   
      
11) 50 g blue cheese , crumbled  
    \[Q U N N O P\]  
    *Here blue cheese is the complete ingredient name, blue is not a form same goes for color in liqueurs. They are not forms, green in green pepper is a form.*  
      
      
12) 6 bay leaves , broken in half for more flavour  
13) 1/2 cup honey-flavored barbecue sauce  
14) 1 spice flavor packet included with corned beef  
      
       
      
    