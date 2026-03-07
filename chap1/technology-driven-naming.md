

技術駆動命名

```
class MemoryStateManager{
    void changeIntValue01(int changeValue){
     intValue01 -= changeValue; 
     if(ntValue01< 0){
      intValue01 = 0; 
      updateState02Flag();        
     }   
    }
}


```

**Python:**
```python
class MemoryStateManager:
    def change_int_value_01(self, change_value: int):
        self.int_value_01 -= change_value 
        if self.int_value_01 < 0:
            self.int_value_01 = 0 
            self.update_state_02_flag()
    # ...
```

**TypeScript:**
```typescript
class MemoryStateManager {
    intValue01: number = 0;

    changeIntValue01(changeValue: number): void {
        this.intValue01 -= changeValue;
        if (this.intValue01 < 0) {
            this.intValue01 = 0;
            this.updateState02Flag();
        }
    }
    // ...
}
```