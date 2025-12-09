# Frontend Troubleshooting Rules

### 1. Add Zustand Package

**Error Message:**
```
Failed to resolve import "zustand"
```

**Solution:**
Run `npm install zustand` command

---

### 2. Loading Import Error

**Error Message:**
```
import { Loading } from '../../components/common/Loading';
Uncaught SyntaxError: The requested module does not provide an export named 'Loading'
```

**Solution:**
Fix to `export const Loading: React.FC<LoadingProps> = ({ message = 'Loading...' }) =>`

---

### 3. 500 Error in Local Environment (Port 3000)

**Error Message:**
```
GET http://localhost:3000/orders 500 (Internal Server Error)
```

**Cause:**
```typescript
useEffect(() => {
  fetchOrders();
}, [fetchOrders]);
// When fetchOrders() is called and backend is unavailable, it fails and screen does not display
```

**Solution:**
Handle axios calls as follows so UI displays normally even when backend is unavailable:
```typescript
const fetchOrders = useCallback(async (params?: { page?: number; size?: number }) => {
  setLoading(true);
  clearError();
  try {
    const response = await orderService.getAll(params);
    setOrders(response.content || []);
  } catch (err: any) {
    // Set empty array even when backend connection fails so UI displays normally
    setOrders([]);
  } finally {
    setLoading(false);
  }
}, [setOrders, setLoading, setError, clearError]);
```

---

### 4. Spring Data REST HAL Format Response Parsing Error

**Error Message:**
```
Uncaught ReferenceError: [entityId] is not defined
```
Or no data displayed in table despite successful API response

**Cause:**
Spring Data REST returns HAL format with _embedded wrapper and ID in _links.self.href:
```json
{
  "_embedded": {
    "orders": [{ ...data, "_links": { "self": { "href": "http://localhost:8084/orders/1" }}}]
  },
  "page": {...}
}
```
Frontend expects flat structure with direct ID field like orderId.

**Solution:**
Add HAL parser to service file that extracts ID from _links.self.href and unwraps _embedded data:
```typescript
const extractIdFromHref = (href: string): number => {
  return parseInt(href.split('/').pop()!, 10);
};

const parseHalResponse = (halData: any) => {
  const items = halData._embedded?.orders?.map((item: any) => ({
    ...item,
    orderId: extractIdFromHref(item._links.self.href)
  })) || [];
  return { content: items, page: halData.page };
};

getAll: async (params) => {
  const { data } = await apiClient.get(BASE_PATH, { params });
  return parseHalResponse(data);
}
```

---

### 5. Wireframe-based Command Components Not Connected to Page

**Error Message:**
- Wireframe generation button always disabled
- Modal onSubmit not working
- TypeScript error "Property 'onSubmit' is missing"

**Cause:**
- Command function not imported from hook
- Modal onSubmit prop missing
- Disabled logic lacking in button

**Solution:**
```typescript
const { modifyOrder, fetchOrders } = useOrder();

<Button
  disabled={!selectedOrderLocal}
  onClick={() => setModifyOrderModalOpen(true)}
>
  Modify Order
</Button>

<ModifyOrderModal
  open={modifyOrderModalOpen}
  onClose={() => setModifyOrderModalOpen(false)}
  onSubmit={async (cmd) => {
    await modifyOrder(cmd);
    fetchOrders();
  }}
  orderId={selectedOrderLocal?.orderId}
/>
```

---

### Best Practices

1. **Always Handle Backend Unavailability** - Set empty array/object in catch block
2. **Parse HAL Responses Correctly** - Extract ID from _links.self.href
3. **Connect Modals Correctly** - Import functions from hooks and pass to onSubmit
4. **Export Components Correctly** - Use named exports with proper TypeScript types
5. **Install Dependencies** - Check package.json and install missing packages
