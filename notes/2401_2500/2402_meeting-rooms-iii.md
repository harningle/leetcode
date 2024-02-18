# 2402. Meeting Rooms III

You are given an integer `n`. There are `n` rooms numbered from $0$ to $n - 1$.

You are given a 2D integer array `meetings` where <code>meetings[i] = [start<sub>i</sub>, end<sub>i</sub>]</code> means that a meeting will be held during the **half-closed** time interval $[start_i, end_i)$. All the values of $start_i$ are unique.

Meetings are allocated to rooms in the following manner:

1. each meeting will take place in the unused room with the **lowest** number
1. if there are no available rooms, the meeting will be delayed until a room becomes free. The delayed meeting should have the **same** duration as the original meeting
1. when a room becomes unused, meetings that have an earlier original **start** time should be given the room.

Return *the __number__ of the room that held the most meetings*. If there are multiple rooms, return *the room with the __lowest__ number*.

A **half-closed interval** $[a, b)$ is the interval between $a$ and $b$ including $a$ and not including $b$.

 
**Example:**


> **Input** `n = 2`, `meetings = [[0, 10], [1, 5], [2, 7], [3, 4]]`
>
> **Output** `0`
>
> **Explanation:**
>
> * at time $0$, both rooms are not being used. The first meeting starts in room $0$.
> * at time $1$, only room $1$ is not being used. The second meeting starts in room $1$.
> * at time $2$, both rooms are being used. The third meeting is delayed.
> * at time $3$, both rooms are being used. The fourth meeting is delayed.
> * at time $5$, the meeting in room $1$ finishes. The third meeting starts in room $1$ for the time period $[5, 10)$.
> * at time $10$, the meetings in both rooms finish. The fourth meeting starts in room $0$ for the time period $[10, 11)$.
>
> Both rooms $0$ and $1$ held $2$ meetings, so we return `0`.


## Priority queue

### Brute force

We have a bunch of meetings, and want to sort them by (original) starting time. We also have a bunch of rooms, and for all rooms with available time before the meeting's (actual) starting time, we pick the first (in terms of index) one. This can be implemented by priority queue.


```cpp
struct earlier {
    bool operator()(const vector<long>& a, const vector<long>& b) {
        return (a[0] > b[0]) || ((a[0] == b[0]) && (a[1] > b[1]));
    }
};

int mostBooked(int n, vector<vector<int>>& meetings) {
    // Pq.: {meeting original start time, delayed start time, actual end time}
    priority_queue<vector<long>,
                   vector<vector<long>>,
                   earlier> remaining_meetings;
    for (auto meeting : meetings) {
        remaining_meetings.push({meeting[0], meeting[0], meeting[1]});
    }

    // Pq.: {available time, room No.}
    priority_queue<vector<long>, vector<vector<long>>, earlier> free_rooms;
    for (int i = 0; i < n; i++) {
        free_rooms.push({0, i});
    }

    // Hashmap: {room idx.: #. of meetings in the room}
    vector<int> cnt(n, 0);

    // While we still have remaining meetings
    vector<long> meeting;        // Current meeting under consideration
    vector<long> earliest_room;  // Earliest possible room
    vector<long> room;           // Possible room with smallest idx,
    while (!remaining_meetings.empty()) {

        // Get the first meeting (by original start time) and first free room
        meeting = remaining_meetings.top();
        earliest_room = free_rooms.top();

        // If any free room, use the one with the smallest room No.
        if (earliest_room[0] <= meeting[1]) {
            // Pq.: {room idx., room available time}
            priority_queue<vector<long>,
                           vector<vector<long>>,
                           earlier> available_rooms;

            // Get all rooms with available time before the meeting
            while ((!free_rooms.empty())
                   && (free_rooms.top()[0] <= meeting[1])) {
                room = free_rooms.top();
                available_rooms.push({room[1], room[0]});
                free_rooms.pop();
            }

            // One with the smallest idx.
            room = available_rooms.top();
            available_rooms.pop();
            cnt[room[0]]++;

            // The room will become free again when the meeting ends, so push
            // back to the pq.
            free_rooms.push({meeting[2], room[0]});

            // Push the unused rooms back
            while (!available_rooms.empty()) {
                room = available_rooms.top();
                free_rooms.push({room[1], room[0]});
                available_rooms.pop();
            }
            remaining_meetings.pop();
        
        // If no free room, delay the meeting
        } else {
            remaining_meetings.pop();
            remaining_meetings.push(
                {
                    meeting[0],
                    earliest_room[0],
                    meeting[2] + earliest_room[0] - meeting[1]
                }
            );
        }
    }
    
    // Find the most busy room
    int ans = 0;
    int freq = 0;
    for (int i = 0; i < n; i++) {
        if (cnt[i] > freq) {
            freq = cnt[i];
            ans = i;
        }
    }
    return ans;
}
```

**Notes**: We need `long` as the start/end time is `int` but after delay it can overflow. So `long`.


### Improvement

There is nothing wrong with the code above, but we can improve a lot:

1. we create room available `priority_queue<vector<long>, vector<vector<long>>, earlier> available_rooms;` for each meeting in the loop
1. we move unused rooms in `available_rooms` back to `rooms`. Each move is $\log n$

Both can be avoided by creating two heaps in the very beginning: rooms in use and rooms available for this meeting. The union of the two is all rooms.

We can further improve by:

3. not creating a priority queue for `meetings`

When we delay a meeting, currently we delete it, and add it back with the modified/delayed time. However, if a meeting is delayed, it's start time and room is determined: the room with the earliest available time. So we can know its room without playing around the queue.


```cpp
int mostBooked(int n, vector<vector<int>>& meetings) {
    // Available rooms for the current meeting. Only need their index
    priority_queue<int> available_rooms;
    for (int i = 0; i < n; i++) {
        available_rooms.push(-i);  // Want min heap so take negative. Can also
    }                              // use a custom comparator as above

    // Rooms that are in use. Need their ending time and room idx.
    priority_queue<vector<long>> rooms_in_use;

    // Sort meetings by their start time
    sort(meetings.begin(), meetings.end());

    // Go through the meetings
    vector<int> cnt(n, 0);
    vector<long> room;
    for (auto& meeting : meetings) {
        // Get all rooms ending before this meeting
        while ((!rooms_in_use.empty())
               && (-rooms_in_use.top()[0] <= meeting[0])) {
            available_rooms.push(rooms_in_use.top()[1]);
            rooms_in_use.pop();
        }

        // If we have available rooms, pick the one with smallest idx.
        if (!available_rooms.empty()) {
            cnt[-available_rooms.top()]++;
            // The room is then being used now. Push it to the queue with
            // the meeting's end time
            rooms_in_use.push({-meeting[1], available_rooms.top()});
            available_rooms.pop();

        // If no available room, then the meeting will be delayed, until
        // there is a free room, which is the room with the earliest end
        // time among thost currently being used
        } else {
            room = rooms_in_use.top();
            cnt[-room[1]]++;
            rooms_in_use.pop();
            rooms_in_use.push({-(meeting[1] -room[0] - meeting[0]), room[1]});
        }
    }

    // Find the most busy room
    int ans = 0;
    int freq = 0;
    for (int i = 0; i < n; i++) {
        if (cnt[i] > freq) {
            freq = cnt[i];
            ans = i;
        }
    }
    return ans;
}
```

