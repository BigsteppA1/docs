---
id: bridge-grandpa
title: GRANDPA Light Client
sidebar_label: GRANDPA Light Client
sidebar_position: 2
---

## Original Process of Proof

A GRANDPA light client needs to verify the validity of a state before accepting a state update. A simple approach is to provide the signatures of the members of the ***GRANDPA*** Authority Set and the supporting evidence along with the state to the light client, so that the light client knows whether there are enough authority members to endorse the state. If more than 2/3 of members sign it(we assume less than 1/3 dishonest Authorities), the light client can accept the state.

## An Improvement Proposal

However, the reality is far from ideal because the number of members of the GRANDPA authority set is so large that 2/3 + 1 signatures and their evidence still account for a large amount of data. Therefore, we need to find ways to reduce this data volume while ensuring sufficient security (the ideal security goal is that the cost of attacking the bridge is equal to the sum of the market value of DOT and ETH).

We can reduce the required number of signatures to reduce the amount of data.

First, we assume that less than 1/3 members in the authority set are dishonest. We can randomly sample 1/3 of the verifier's signatures; the light client can be confident that the state is valid if all the sampled signatures are the same.

### Steps of Verification

1. The light client has an confirmed Merkle root of validator public keys ***rval***;
2. Then the light client receives a message containing  ***S***, ***b***, ***sig***, ***p***; 
    - ***S***: State
    - ***b***: A bit field flagging the signing authority member
    - ***sig***: A signature for 𝑆 from any member in the authority set
    - ***p***: The Merkle proof of the public key of the above member
    
    
3. The light client uses the received information to verify that the State  ***S***  is endorsed by this authority member;
4. The light client requests the following information from the relayer: the Merkle proof ***P*** of every public key of the randomly selected members from the authority set who signed;
5. The light client then verifies the requested information to ensure having collected enough signatures.

### Security Analysis

1. Random Number Generation
    
    In Step 4, there is a random choice involved. We need to hide randomness unpredictable to the attackers. We can use a number ***txhash*** as a seed for random number generation. This ***txhash*** denotes the transaction hash when the potential attack happens which can not be predicted by the attacker.
    
2. Cost Analysis of Attacks
    
    Suppose an evil-intentioned authority member wants to submit a problematic state, what is the probability of the state being accepted?
    
    ![Authority set, Signers, and Attackers](../../../assets/bridge_knowledge_base_grandpa_01.png)
    
As the figure shows, ***N*** is the set of the whole authority set, ***b*** (>2/3) denotes those who sign, and the bad authorities is a subset of ***b***. **Bad authorities** account for less that 1/2 of ***b***. So in the worst case, all the randomly selected members of authority set are ***bad***, whose probability is less than ***(1/2)<sup>k</sup> ***. Only when this happens, the attack can be successful. Every authority member is required to stake some assets ***minsupport***. Then we have the expected tries of a successful attack *** E<sub>tries</sub> ***

*** E<sub>tries</sub> > 2<sup>k</sup> ***

and the expected cost *** E<sub>cost</sub> ***

*** E<sub>cost</sub> > minsupport x 2 <sup>k</sup> ***
