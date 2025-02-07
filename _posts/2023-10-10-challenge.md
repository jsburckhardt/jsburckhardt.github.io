---
layout: post
title: Different challenges in golang
date: 2023-10-10
---

Rate limiter

```go
type RateLimiter struct {
    mu         sync.Mutex
    limits     map[string]*UserLimit
    maxRequests int
    window      time.Duration
}

type UserLimit struct {
    count     int
    windowStart time.Time
}

func NewRateLimiter(max int, window time.Duration) *RateLimiter {
    return &RateLimiter{
        limits:      make(map[string]*UserLimit),
        maxRequests: max,
        window:      window,
    }
}

func (rl *RateLimiter) Allow(userID string) bool {
    rl.mu.Lock()
    defer rl.mu.Unlock()

    now := time.Now()
    if ul, exists := rl.limits[userID]; exists {
        // Check if window expired
        if now.Sub(ul.windowStart) > rl.window {
            ul.windowStart = now
            ul.count = 1
            return true
        }
        // If within window and count is below limit
        if ul.count < rl.maxRequests {
            ul.count++
            return true
        }
        return false
    }
    // New user: initialize
    rl.limits[userID] = &UserLimit{
        count:       1,
        windowStart: now,
    }
    return true
}
```

Voting
```go
// Define a Vote structure
type Vote struct {
    UserID    string
    OptionID  string
    Timestamp time.Time
}

// Imagine a function that records or updates a vote
func RecordVote(vote Vote) error {
    // Pseudocode: check if user has an existing vote
    existing, err := db.GetVote(vote.UserID)
    if err != nil {
        return err
    }
    if existing != nil {
        // update the vote if different
        if existing.OptionID != vote.OptionID {
            db.UpdateVote(vote)
        }
    } else {
        db.InsertVote(vote)
    }
    // Recompute and publish new vote counts
    publishVoteCounts()
    return nil
}
```

```string
func addStrings(num1, num2 string) string {
    i, j := len(num1)-1, len(num2)-1
    carry := 0
    var result []byte

    for i >= 0 || j >= 0 || carry > 0 {
        sum := carry
        if i >= 0 {
            sum += int(num1[i] - '0')
            i--
        }
        if j >= 0 {
            sum += int(num2[j] - '0')
            j--
        }
        carry = sum / 10
        digit := sum % 10
        result = append(result, byte(digit)+'0')
    }

    // Reverse result
    for i, n := 0, len(result); i < n/2; i++ {
        result[i], result[n-1-i] = result[n-1-i], result[i]
    }
    return string(result)
}
```
