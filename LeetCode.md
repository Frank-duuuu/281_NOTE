## 1029 two city scheduling
greedy method: sort the vector based on the difference of Cost_cityB - cost_cityA

smaller number should send to cityB as their city B cost are smaller than city A cost compared with others
remember that half of the people should go to city B while the other half to city B

````C++
int twoCitySchedCost(vector<vector<int>>& costs) {
        sort(costs.begin(),costs.end(),[](vector<int> &v1,vector<int> &v2){
        return v1.back() - v1.front() < v2.back() - v2.front();
    });
    
    int total_cost = 0;
    for(size_t i = 0;i < costs.size();i++){
        //element in the front should send to city B ,as they have relatively
        //small cost_B - cost_A,
        if(i < costs.size() / 2)
            total_cost += costs[i].back();
        else
            total_cost += costs[i].front();
    }
    
    return total_cost;
    }
````

## count number of rectangle, eecs281 question

<img width="800" alt="image" src="https://user-images.githubusercontent.com/81163933/174462110-b9639612-abba-4bf0-a411-1199a8102adb.png">

IDEA: using hash map, counting how many vertical edges you can have. for example (1,1) and (1,2) form one vertical edges, we can stores pair( 1, 2), into the hash map, using this key, we can know how many vertical edges with same length we have and thus calculate how many rectangles we can have

````C++
int count_rectangles(vector<Point> &points) {
map<pair<int, int>, int> lines;
int count = 0;
for (auto &p1 : points) {
        for (auto &p2 : points) {
            if (p1.x == p2.x && p1.y < p2.y) {
                pair<int, int> vertical_line{ p1.y, p2.y };
                count += lines[vertical_line]++;
             }
        }
}
return count;
}
````

## Leetcode 752. open the lock

we have a helper function that can return all sequences that can be attained in a single turn from the original sequence orig_seq.

`Idea:`
        
1.try different combination using DFS, we start at "0000"
2. using unordered_set<string> dead_end and visited to check whether we reach the dead end and mark sequence as visited.

`Mistakes I made`
1. we can not check the dead end in the innest for loop and return -1. if we do so, we have not tried all possiblities in that layer, simply             drop that solution and don't push to the queue if it has been visited or it is dead end
 2. we need cur_layer_size variable to make sure we look into every sequences within same layer in one pass of while loop
 
````C++
        vector<string> single_turn_seqs(string orig_seq) {
         vector<string> result;
        for (int i = 0; i < 4; ++i) {
             string temp = orig_seq;
             temp[i] = (orig_seq[i] - '0' + 1) % 10 + '0';
            result.push_back(temp);
            temp[i] = (orig_seq[i] - '0' - 1 + 10) % 10 + '0';
            result.push_back(temp);
    }
    return result;
        }
int openLock(vector<string>& deadends, string target) {
    unordered_set<string> dead(deadends.begin(),deadends.end());
    unordered_set<string> visited;
    queue<string> my_q; // dfs
    int result = 0;
    my_q.push("0000");
    
    if(dead.find("0000") != dead.end()){return -1;}
    
    while(!my_q.empty()){
        size_t cur_layer_size = my_q.size();
        
        for(size_t i = 0;i < cur_layer_size;i++){
            string cur_s = my_q.front();
            my_q.pop();
            //target?
            if(cur_s == target){return result;}

            auto possible = single_turn_seqs(cur_s);
            
            //for all possible sequences, check visited? then check deadend? if not result++
            for(auto &i: possible){
                
                //visited?
                if(visited.find(i) == visited.end()
                   && dead.find(i) == dead.end()){
                    visited.insert(i); //mark as visited
                    my_q.push(i);
                }
                        
            }//for
        }//current layer for

        result++;
        
    }//while
    
    return -1;
    
}
````
               
## Leetcode 743 Network Delay time
                                                 
* Idea : using dijstra's algorithm
* using min-heap , priority queue, need dist vector to record all distance to that nodes, need hash map to search for certain edges

````C++
  //   total length | node
//
int networkDelayTime(vector<vector<int>>& times, int n, int k) {
    priority_queue<pair<int,int>,vector<pair<int,int>>,greater<pair<int,int>>> my_q;
    unordered_map<int, vector<pair<int,int>>> my_map;
    vector<int> dist(n + 1, std::numeric_limits<int>::max());
    
    for(auto & i: times){
        my_map[i[0]].push_back(make_pair(i[1],i[2]));
    }
    
    dist[k] = 0;
    my_q.push(make_pair(0,k));//start from number k
    
    while(!my_q.empty()){
        int cur_node = my_q.top().second;
        my_q.pop();
        
        //find for this cur_node, find and update all reachable nodes
        
        for(auto it = my_map[cur_node].begin(); it != my_map[cur_node].end();++it){
            //new_length to add = it->second
            //new_node  = it->first
            
            int new_node = it->first; int new_length_to_add = it->second;
            
            if(dist[cur_node] + new_length_to_add < dist[new_node]){
                dist[new_node] = dist[cur_node] + new_length_to_add;
                my_q.push(make_pair(dist[new_node],new_node));
            }
            
        }
    }
    
    //find max distance
    int max = numeric_limits<int>::min();
    
    for(size_t i = 1;i < dist.size();i++){if(dist[i] > max){max = dist[i];}}
    
    if(max == numeric_limits<int>::max())
        return -1;
    else
        return max;
  
        
}              
````

## Zero Sum Game 0. 281  problem
![IMG_1D14B45E67DD-1](https://user-images.githubusercontent.com/81163933/174652052-b9ae94a3-5907-4cd4-a87f-6751c00190de.jpeg)

* Idea: using hase_set to record the trends of the cumulative sum of sequence, if there are two points a and b that cumulative sum are the same, this means that the sequence between point a and point b must be zero
````C++
                bool zero_contiguous_sum(vector <int> &nums){
                        if(nums.empty()){return false;}
                        
                        unordered_set <int> my_set;
                        int cur_sum = 0;
                        my_set.insert(cur_sum);
                        for(auto &i : nums){
                                cur_sum += i;
                                if(my_set.find(cur_sum) != my_set.end()){return true; //same cumulative sum point found}
                                else{my_set.insert(cur_sum);}
                        }
                return false;
                
                }
````
                
                
## 128. Generate Parentheses
                
* Method I: back tracking.
                
* Mistakes I have: when doing recursive function you can do func(a + 1) but not func(++a). You are not really increasing this element, but rather make    it seems like increasing from the perspective of next recursive function.(pass the increased value in but not modify value from current function)
                
````C++
void back_track(vector<string> &result,string cur, int num_L,int num_R, int size){
        //base case
        if(cur.size() == size*2){
            result.push_back(cur);
            return;
        }
        
        //decision  I have
        if(num_L < size){
            //add a Left parentheses
            back_track(result,cur+'(',num_L+1,num_R,size);
        }
        
        if(num_L > num_R){
            //add a right parentheses
 
            back_track(result,cur+')',num_L,num_R + 1,size);
        }
    
    }
    
    
    vector<string> generateParenthesis(int n) {
        vector<string> my_vec;
        back_track(my_vec,"",0,0,n);
        return my_vec;
    }                
````
                
                
     
