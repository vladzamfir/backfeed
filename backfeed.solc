contract Backfeed {	

    uint contribution_fee;
    uint contribution_counter;
    uint BIG_NUM;


    function Backfeed{
        contribution_fee = 100;
        contribution_counter = 0;
        BIG_NUM = 2**20
    }


    struct community {
        string name;
        
        uint total_reputation;
        mapping (address => uint) reputation;

        uint total_tokens;
        mapping (address => uint) balance;

        uint community_event_counter;

        //N.b. that we must keep track of the medians on a by-community basis
        mapping (hash => uint) max_median;
        mapping (hash => uint) last_median;
    }


    mapping (string => community) communities;

    function found_community( string name, mapping (address => uint) initial_rep, mapping (address => uint) initial_coins) {

        //if the community already exists, exit
        if(communities[name]){
            return "name already registered";
        }

        //otherwise, we create the new community
        community new_community;

        new_community.name = name;
        
        new_community.reputation = initial_rep;
        new_community.total_reputation = 0;
        for (addr in initial_rep){ //this loop is not proper solidity
            new_community.total_reputation += initial_rep[addr];
        }

        new_community.balance = initial_coins;
        new_community.total_tokens = 0;
        for (addr in initial_coins){ //this loop is not proper solidity
            new_community.total_tokens += initial_coins[addr];
        }

        new_community.community_event_counter = 0;

        communities[name] = new_community;
    }


    struct contribution {
        hash cont_hash;
        mapping (address => uint) contributors; //might need to be normalized later
        unit counter;
    }

    mapping (hash => contribution) contributions;

    function contribute(contribution c) {
        //atm we're paying the contribution fee in ether, because I haven't bothered to initialize it with a backfeed community, yet
        if(msg.value < contribution_fee) {
            return "expected contribution fee"; 
        }

        // if the contribution already exists, exit
        if(contributions[c.cont_hash]){
            return;
        }


        contribution new_contribution;
        new_contribution = c;
        contribution_counter += 1
        new_contribution.counter = contribution_counter;

        contributions[c.cont_hash] = new_contribution;
    }


    struct evaluation {
        hash cont_hash;
        unit score;  //in general we might allow any distribution on [0,infty]
        address evaluator;
        unit evaluator_rep;
        unit evaluator_stake;
        unit counter;
        string community_tag; //I only have one community tag, because otherwise we'd need to have an array of evaluator rep and stake
        string[BIG_NUM] semantic_tags; //these maybe should be weighted
    }

    //mapping from community names and contribution hashes to num_evaluations and evaluations
    // may not be proper solidity
    mapping ( (string, hash) => (uint, evaluation[BIG_NUM]) ) evaluations;

    //we'll have two or three global parameters to stretch this function
    function decay(uint x, uint y){
        return 1/(1 + exp((x-y)**2)); 
    }

    //the reputation at stake from evaluation e2 will be partially redistributed to the evaluator of e1 
    function redistribution(evaluation e1, evaluation e2){
        s1 = e1.evaluator_stake;
        s2 = e2.evaluator_stake;
        v1 = e1.score;
        v2 = e2.score;


        num_evals = evaluations[(e1.community_tag, e1.cont_hash)][0];
        evals = evaluations[(e1.community_tag, e1.cont_hash)][1];

        normalizing_factor = 0;
        for (var i = 0; i < num_evals; i++){
            normalizing_factor += decay(evals[i].score,v2)*evals[i].evaluator_stake;
        }

        return s1*s2*decay(v1,v2)/normalizing_factor;
    }
    

    function evaluate(evaluation e) {

        evaluation new_evaluation;
        new_evaluation = e;

        new_evaluation.evaluator_rep = communities[e.community_tag].reputation[e.evaluator];

        // boolean condition checks that the stake is correct - some protocols will have minimum, or a prescribed amount
        // THIS FUNCTION IS NOT YET SPECIFIED 
        if (invalid_stake(new_evaluation.evaluator_rep, e.evaluator_stake))  {
            return "invalid stake";
        }

        communities[e.community_tag].community_event_counter += 1;
        new_evaluation.counter = communities[e.community_tag].community_event_counter;

        num_evals = evaluations[(e1.community_tag, e1.cont_hash)][0] + 1;
        evaluations[(e1.community_tag, e1.cont_hash)][0] = num_evals;

        evaluations[(e.community_tag, e.cont_hash)][1][num_evals] = new_evaluation;


        //this part is not yet fully specified - this is basically crappy pseudocode
        median = MEDIAN(evaluations[(e.community_tag, e.cont_hash)], (0, remaining reputation))

        //I need to initialize the max_median value to 0
        //this issues tokens based on the evaluations
        //I need issue coins to all of the contributors of a contribution in proportion to their weight
        if median > communities[e.community_tag].max_median[e.cont_hash]{
            communities[e.community_tag].balance[contributions[e.cont_hash].contributor] += median - communities[e.community_tag].max_median[e.cont_hash];   
        }

        // f(reputation) scales the change in reputation in order to slow down low reputation entities\
        //THIS FUNCTION IS NOT YET SPECIFIED
        communities[e.community_tag].reputation[contributions[e.cont_hash].contributor] += 
        (median - communities[e.community_tag].last_median[e.cont_hash]) * f(communities[e.community_tag].reputation[contributions[e.cont_hash].contributor]) 


        communities[e.community_tag].reputation[e.evaluator] -= e.evaluator_stake;

        num_evals = evaluations[(e.community_tag, e.cont_hash)][0];
        evals = evaluations[(e.community_tag, e.cont_hash)][1];
        for (var i = 0; i < num_evals - 1; i++){
            communities[e.community_tag].reputation[evals[i].evaluator] += redistribution(evals[i], new_evaluation);
        }
    }
}