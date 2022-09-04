# Zig AStar

This repository contains a simple implementation of the A-Star algorithm in Zig.

This is not intended to be fancy. However, it does take a user-defined
position type and distance function, and does not assume anything about the
space that the pathfinding takes place in.

Instead the algorithm expects the user to drive it, tracking paths and asking the user
for the neighbors of a particular location. When the algorithm finds a path to the end
location it will report that it is done. This is similar to an iterator, and avoids
requiring any kind of user defined map type or "neighbors" function to be provided.

The implementation uses the std.ArrayList and std.PriorityQueue, and takes an allocator
from the user.


See below for an example use. Notice that the user code drives the search by
calling 'step', and feeding back the requested slice of neighbor positions.
```zig

const SimplePos = struct {
    x: isize,
    y: isize,

    pub fn init(x: isize, y: isize) SimplePos {
        return SimplePos{ .x = x, .y = y };
    }
};

fn simple_distance(start: SimplePos, end: SimplePos) usize {
    const x_dist = std.math.absInt(start.x - end.x) catch unreachable;
    const y_dist = std.math.absInt(start.y - end.y) catch unreachable;
    return @intCast(usize, std.math.min(x_dist, y_dist));
}

pub main() void {
    const allocator = std.heap.page_allocator;

    const PathFinder = Astar(SimplePos, simple_distance);

    var neighbors = ArrayList(SimplePos).init(allocator);
    defer neighbors.deinit();
    
    const start = SimplePos.init(0, 0);
    const end = SimplePos.init(4, 4);
    
    var result = try finder.pathFind(start, end);
    while (result == .neighbors) {
        const pos = result.neighbors;

        neighbors.clearRetainingCapacity();

        const offsets: [3]isize = .{ -1, 0, 1 };
        for (offsets) |offset_x| {
            for (offsets) |offset_y| {
                const new_x = pos.x + offset_x;
                const new_y = pos.y + offset_y;
                
                // User defined validity function 'IsValid' not shown.
                if (!IsValid(new_x, new_y)) {
                    continue;
                }
                const next_pos = SimplePos.init(new_x, new_y);
                try neighbors.append(next_pos);
            }
        }

        result = try finder.step(neighbors.items);
    }
    
    // The 'result' variable is now either .done, with the Path structure containing the
    // path from start to end, or .no_path indicating that there is no valid path.
    switch (result) {
        .no_path => {
            // Error
        },
        
        .done => |path| {
            // Path from start to end of type Path(SimplePos).
        },
        
        .neighbors => {
            unreachable
        },
    }
}
```
