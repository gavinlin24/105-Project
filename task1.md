# Task 1
## Code
```python
"""
ChatGPT was used to help create this program.

Prompts used:

- I need to write python code that checks if two regular properties are consistent. How would you approach this?
- How can you represent a DFA as an input string?
- How can I implement the intersection of 2 DFAs?
- How can I implement a function to determine if the language of a DFA is empty?

"""
from collections import deque

class DFA:
    """
    A class used to represent a DFA.

    Attributes
    ----------
    states : set
        A set containing all states of the DFA.
    alphabet : set
        A set of symbols that form the alphabet of the DFA.
    start_state : str
        The initial state of the DFA.
    accept_states : set
        A set of states that are considered accepting (final) states.
    transitions : dict
        A dictionary mapping (state, symbol) pairs to the next state.

    Methods
    -------
    get_state(state, symbol)
        Returns the next state given the current state and input symbol.
    construct_dfa(dfa_string)
        Parses a formatted string to construct a DFA.
    """

    def __init__(self, states, alphabet, start_state, accept_states, transitions):
        """
        Parameters
        ----------
        states : set
            A set containing all states of the DFA.
        alphabet : set
            A set of symbols that form the alphabet of the DFA.
        start_state : str
            The initial state of the DFA.
        accept_states : set
            A set of states that are considered accepting (final) states.
        transitions : dict
            A dictionary mapping (state, symbol) pairs to the next state.
        """
        self.states = set(states)
        self.alphabet = set(alphabet)
        self.start_state = start_state
        self.accept_states = set(accept_states)
        self.transitions = transitions

    @staticmethod
    def construct_dfa(dfa_string):
        """
        Constructs a DFA object from a formatted string.

        DFA strings are formatted by their 5-tuple definition seperated by a semi-colon:

        states ; alphabet ; start_state ; accept_states ; transitions

        transitions:
            - format: current_state,input->destination
            - each transition seperated by ;
        
        Parameters
        ----------
        dfa_string : str
            A string representing some DFA.
        """
        parts = dfa_string.split(";")
        states = parts[0].split(",")
        alphabet = parts[1].split(",")
        start_state = parts[2]
        accept_states = parts[3].split(",") if parts[3] else []
        transitions = {}

        for i in range(4, len(parts)):
            if parts[i]:
                src, rest = parts[i].split(",")
                symbol, dst = rest.split("->")
                transitions[(src, symbol)] = dst
            
        return DFA(states, alphabet, start_state, accept_states, transitions)

    def get_state(self, state, symbol):
        """
        Helper function to find destination state given current state and input symbol.
        
        Parameters
        ----------
        state : str
            The current state of the DFA.
        symbol: str
            The current input symbol being read.
        """
        return self.transitions.get((state, symbol))

def combine_dfa(dfa1, dfa2):
    """
    Returns a new DFA object of the intersection of 2 DFAs.
    
    Parameters
    ----------
    dfa1 : DFA
        The first DFA object.
    dfa2: DFA
        The second DFA object.
    """
    new_states = set()
    new_transitions = {}
    new_accept_states = set()
    
    queue = deque()
    start_state = (dfa1.start_state, dfa2.start_state)
    queue.append(start_state)
    new_states.add(start_state)

    while queue:
        state1, state2 = queue.popleft()
        
        if state1 in dfa1.accept_states and state2 in dfa2.accept_states:
            new_accept_states.add((state1, state2))
        
        for symbol in dfa1.alphabet and dfa2.alphabet:
            next1 = dfa1.get_state(state1, symbol)
            next2 = dfa2.get_state(state2, symbol)

            if next1 and next2:
                new_state = (next1, next2)
                if new_state not in new_states:
                    new_states.add(new_state)
                    queue.append(new_state)
                new_transitions[((state1, state2), symbol)] = new_state
    
    return DFA(new_states, dfa1.alphabet & dfa2.alphabet, start_state, new_accept_states, new_transitions)

def is_empty(dfa):
    """
    Checks if the language of a DFA is empty using BFS.

    ie. From the starting state, if at any point an accept state can be reached,
        return False.

        Otherwise if all states of the DFA are visited and no accept state is reached,
        return True.
    
    Parameters
    ----------
    dfa : DFA
        A DFA object.
    """
    visited = set()
    queue = deque([dfa.start_state])
    
    while queue:
        state = queue.popleft()
        if state in dfa.accept_states:
            return False
        
        if state in visited:
            continue
        visited.add(state)

        for symbol in dfa.alphabet:
            next_state = dfa.get_state(state, symbol)
            if next_state and next_state not in visited:
                queue.append(next_state)

    return True

def is_consistent(dfa1_str, dfa2_str):
    """
    Main function that checks whether 2 DFA's are consistent.

        - constructs a DFA object for both string representations.
        - constructs another DFA object representing the intersection of the 2 DFAs.
        - checks whether any language is recognized by the intersection.
        - intersection is empty => DFA's not consistent.
        - intersection is non-empty => DFA's consistent.
    
    Parameters
    ----------
    dfa1_str : str
        The first string representing some DFA.
    dfa2_str : str
        The second string representing some DFA.
    """
    dfa1 = DFA.construct_dfa(dfa1_str)
    dfa2 = DFA.construct_dfa(dfa2_str)

    intersect_dfa = combine_dfa(dfa1, dfa2)
    return not is_empty(intersect_dfa)

# Example DFAs
w0 = "qstart,q0,q1;0,1;qstart;q0;qstart,0->q0;qstart,1->q1;q0,0->q0;q0,1->q0;q1,0->q1;q1,1->q1"
w1 = "rstart,r0,r1;0,1;rstart;r1;rstart,0->r0;rstart,1->r1;r0,0->r0;r0,1->r0;r1,0->r1;r1,1->r1"

x0 = "q0,q1;0,1;q0;q0;q0,1->q1;q0,0->q1;q1,1->q0;q1,0->q0"
x1 = "r0,r1;0,1;r0;r1;r0,1->r1;r0,0->r0;r1,1->r1;r1,0->r0"

print(is_consistent(w0, w1))
print(is_consistent(x0, x1))
```
## String Representation of DFAs
DFAs are represented by the 5-tuple definition in the program each seperated by a semi-colon and listed items seperated by commas. (This representation was created by ChatGPT)

* states ; alphabet ; starting-state ; accepting-states ; transitions

More specifically,

* "state1, ... , stateN ; char1, ... , charN ; starting-state ; accept1, ... , acceptN ; transition1 ; ... ; transitionN"

Note the transitions are formatted like this:

* "current-state, input-character->destination"

Multiple transitions are seperated by semi-colons:

* "current-state, input-character->destination;current-state, input-character->destination"

For example,

Let DFA M = {Q, Σ, δ, q0, F}.
* Q = {q0}
* Σ = {0, 1}
* δ(q0, 0) = q0, δ(q0, 1) = q0
* F = {q0}

Then the string representation for M is:

"q0;0,1;q0;q0;q0,0->q0;q0,1->q0"
## Running the Code

### w0 and w1
In the program, w0 and w1 are represented in string format by the following:
```python
w0 = "qstart,q0,q1;0,1;qstart;q0;qstart,0->q0;qstart,1->q1;q0,0->q0;q0,1->q0;q1,0->q1;q1,1->q1"
w1 = "rstart,r0,r1;0,1;rstart;r1;rstart,0->r0;rstart,1->r1;r0,0->r0;r0,1->r0;r1,0->r1;r1,1->r1"
```
