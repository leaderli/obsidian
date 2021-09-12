# reflect


## 反射工具类创建数组
```java
int arr[] = (int[])Array.newInstance(int.class, 5);
Array.set(arr, 0, 5);
Array.set(arr, 1, 1);
Array.set(arr, 2, 9);
Array.set(arr, 3, 3);
Array.set(arr, 4, 7);
System.out.print("The array elements are: ");
for(int i: arr) {
    System.out.print(i + " ");
}
```