# LeetCode刷题框架

## 网格搜索类型

```Java
public void dfs(char[][] board,int x,int y){
    //先判断越界以及该点是否访问和其他逻辑
    if(x<0 || x>=board.length || y<0 || y>=board[x].length){
        return ..;
    }
    //利用数组，四方搜索
    int[] arr = new int[]{-1,0,1,0,-1};
    //若是全图搜索,这是标志
    vis[x][y]=true;
    //若是回溯，需要搜索多次，先改值
    char ch = board[x][y];
    board[x][y]='#';
    for(int i=0;i<4;i++){
        dfs(board,x+arr[i],y+arr[i+1]);
    }
    //再改回来（在四个方向搜索完改回来）
    //若后面有依赖board[x][y]值的，一定要先改回来了再进行
    board[x][y]=tmp;
    return ..;
}
```

## 链表

### 反转链表（1、2、k）

```Java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        //递归思路是先进行一次k个反转，然后递归反转剩下的并且连接起来
        ListNode tail = head;
        int num=0;
        while(num!=k && tail!=null){
            tail=tail.next;
            num++;
        }
        if(num!=k){
            return head;
        }
        ListNode ans = reverse3(head,tail);
        head.next = reverseKGroup(head.next,k);
        return ans;
    }
    //一次反转函数传入参数为head，tail，结束只要将head和tail相连接就不会出错.其中只是将null换成了tail
    
    //头插法
    public ListNode reverse(ListNode root,ListNode tail){
        ListNode ans=root,cur=root.next;
        while(cur!=tail){
            root.next=cur.next;
            cur.next = ans;
            ans = cur;
            cur = root.next;
        }
        return ans;
    }
    //递归，注意0，1个节点
    public ListNode reverse2(ListNode root,ListNode tail){
        if(root!=tail && root.next==tail){
            return root;
        }
        ListNode ans = reverse2(root.next,tail);
        root.next.next = root;
        root.next = tail;
        return ans;
    }
    //双指针
    public ListNode reverse3(ListNode root,ListNode tail){
        ListNode pre=null,cur=root;
        while(cur!=tail){
            ListNode tmp = cur.next;
            cur.next=pre;
            pre = cur;
            cur = tmp;
        }
        root.next = tail;
        return pre;
    }
}
```

### 快慢指针

-    寻找`环以及起点`
-    寻找`中点`（延申`回文`、`链表归并排序`）

## 双指针

-    合并有序链表（2，k）
-    判断交点（遍历两次互相交换头节点）

     
