0. QA: 
_auctionElapsedSeconds is either = uint64(0) or uint64(block.timestamp - _rngCompletedAt)

The "problem" here is that since the condition is:

block.timestamp < _rngCompletedAt

It means that the equal in block.timestamp >= _rngCompletedAt is not covered, which means that uint64(block.timestamp - _rngCompletedAt) can be zero, but uint64(0) is also zero. So in both cases _auctionElapsedSeconds can be zero. Was this intended or not?

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L139

