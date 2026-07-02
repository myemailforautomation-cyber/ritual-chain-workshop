# Privacy-Preserving AI Bounty Judge

## Commit-Reveal Lifecycle

1. The bounty owner creates a bounty and deposits the reward.
2. Participants compute a commitment:

   keccak256(answer, salt, msg.sender, bountyId)

3. Participants submit only the commitment hash during the submission phase.
4. After submissions close, participants reveal their answer and salt.
5. The contract verifies that the computed hash matches the stored commitment.
6. Only successfully revealed answers are eligible for AI judging.
7. The bounty owner calls `judgeAll()` to evaluate the revealed submissions.
8. The winner is finalized and receives the reward.

---

## Test Plan

### Valid Commitment
- User submits a valid commitment hash.
- Expected: submission succeeds.

### Valid Reveal
- User reveals using the correct answer and salt.
- Expected: reveal succeeds and `revealed = true`.

### Invalid Salt
- User reveals with the wrong salt.
- Expected: transaction reverts with `invalid reveal`.

### Invalid Answer
- User reveals with a different answer.
- Expected: transaction reverts with `invalid reveal`.

### Double Reveal
- User attempts to reveal twice.
- Expected: transaction reverts with `already revealed`.

### Judge Without Revealed Answers
- Owner calls `judgeAll()` before any valid reveals.
- Expected: transaction reverts with `no revealed submissions`.

### Finalize Unrevealed Winner
- Owner selects a submission that was never revealed.
- Expected: transaction reverts with `winner must be revealed`.

---

## Architecture Note

### Public Information
- Bounty title and rubric
- Reward amount
- Commitment hashes
- AI review results
- Final winner

### Hidden Information
- User answers during the submission phase
- User salts used to generate commitments

### AI Responsibilities
- Evaluate revealed submissions in batches
- Produce rankings and feedback according to the rubric

### Human Responsibilities
- Define bounty requirements
- Review AI outputs if necessary
- Finalize the winner

### Commit-Reveal Flow

1. User computes:

   keccak256(answer, salt, msg.sender, bountyId)

2. User submits only the commitment.

3. User later reveals the answer and salt.

4. The contract verifies the commitment.

5. Only revealed answers can be judged by the AI.

---

## Reflection Question

In a bounty system, the bounty details, rewards, and final results should be public so that everyone can verify the process. Participants' answers should remain hidden during the submission period to prevent copying and unfair advantages. The salts used for commitments should also stay private because they protect the original answers. AI should help evaluate submissions consistently and efficiently based on the provided rubric. Humans should still make the final decision when subjective judgment is needed or when reviewing AI outputs. Combining AI and human oversight creates a fairer and more transparent system. The commit-reveal approach helps protect participants while maintaining trust in the final results.