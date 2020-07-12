#### useCallback
1. useCallback的回调函数在组件每次render时都会被创建（不管依赖数组是否变化），useCallback的性能优化不体现于是否创建新函数。
2. 当依赖函数变化时，将返回新创建的内联函数，否则返回缓存的旧函数。在子组件使用React.memo（props值不变，则不重新render子组件）的情况下，useCallback可以避免“父组件需要重新render，但依赖数组不变”场景下，子组件的render。
3. 许多文章都有提到，创建内联函数是廉价的，这也不是useCallback致力于优化的点，考虑到inputs的新旧比较本身就有一定花销，与React.memo搭配才是使用useCallback的正确姿势。
4. 所有回调函数中引用的值都应该出现在依赖项数组中。
