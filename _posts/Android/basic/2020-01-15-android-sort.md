---
layout: post
title:  "[Android] 정렬 - Comparator, Comparable, 버블 정렬, 선택 정렬, 퀵 정"
categories: Android
tags: CS
author: TaeHyungK
---

* content
{:toc}

### Comparator와 Comparable

 - Comparator
   - 기본 정렬(Integer, Double - 오름차순 / String - 사전순)과 다르게 정렬하고 싶을 때 사용하는 추상 클래스
   - compare() 메소드를 통해 비교
     > 음수 또는 0을 반환할 경우 두 비교 대상의 위치가 그대로 유지
     > 양수를 리턴할 경우 두 비교 대상의 위치를 바꿈
   ```java
   // 내림차순 정렬
   public class ReverseNumber implements Comparator<Integer> {
       @Override
       public int compare(Integer o1, Integer o2){
           return o2 - o1;
       }
   }
   ```

 - Comparable
   - 기본 정렬을 어떤 기준으로 하느냐를 커스텀할 때에 사용하는 추상 클래스
   - compareTo() 메소드를 통해 비교
     > 음수 또는 0을 반환할 경우 두 비교 대상의 위치가 그대로 유지
     > 양수를 리턴할 경우 두 비교 대상의 위치를 바꿈








### 정렬의 종류

 - 버블 정렬
   - 인접한 두 원소를 비교하여 위치를 바꿔가며 정렬하는 방식
   - O(N^2)의 시간 복잡도를 가짐
   ```java
   private void bubbleSort(int[] input) {
       int length = input.length;
       int tmp;
           
       for (int i = 1; i < length; i++) {
           for (int j = 0; j < length - i; j++) {
               if (input[j] > input[j + 1]) {
                   tmp = input[j];
                   input[j] = input[j + 1];
                   input[j + 1] = tmp;
               }
           }
       }
   
       for (int i = 0; i < length; i++) {
           System.out.println("bublleSort() " + items[i]);
       }
   }
   ```
 
 - 선택 정렬
   - 하나의 값을 선택해 배열을 돌며 가장 작은 값과 위치를 바꿔가며 정렬하는 방식
   - O(N^2)의 시간 복잡도를 가짐
   ```java
   private void selectionSort(int[] input) {
       int length = input.length;
       int tmp;
       for (int i = 0; i < length - 1; i++) {
           for (int j = i + 1; j < length - 1; j++) {
               if (input[i] > input[j]) {
                   tmp = input[i];
                   input[i] = input[j];
                   input[j] = tmp;
               }
           }
       }
   } 
   ```
   
 - 퀵 정렬
   - 배열을 기준값(pivot)을 기준으로 분할하여 기준값보다 작은 원소는 왼쪽으로, 큰 원소는 오른쪽으로 정렬하여 분할된 배열의 크기가 0이나 1이 될 때까지 순환 호출하여 분할하는 작업을 반복하여 정렬하는 방식
   - 평균 O(nLog2N)의 시간 복잡도를 가지나 최악의 경우(pivot에 따라 달라짐) O(N^2)의 시간복잡도를 가짐
   ```java
   public static void main(String[] args) {
       int[] input = {10, 32, 55, 1 ,2, 59}
       quickSort(input, 0, input.length - 1);
   }
   private void quickSort(int[] input, int left, int right) {
       if (left < right) {
           // 값을 비교하고 로우와 하이를 이동시키면서 값의 교환이 이루어지는 함수
           int pivotNewIndex = partition(input, left, right);
           // 재귀 및 분할
           quickSort(input, left, pivotNewIndex - 1);
           quickSort(input, pivotNewIndex + 1, right);
       }
   }
   private int partition(int[] input, int left, int right) {
       int pivot = input[(left + right) / 2];
     
       while (left < right) {
           while ((input[left] < pivot]) && (left < right)) left++;
           while ((input[right] > pivot) && (left > right)) right--;
       
           if (left < right) {
               int temp = input[left];
               input[left] = input[right];
               input[right] = temp;
           }
       }
       return left;
   }
   ```
