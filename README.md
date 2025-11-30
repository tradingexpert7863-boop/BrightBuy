import React, { useState, useEffect } from "react";

// Simple single-file React component for an e‑commerce demo
// Uses Tailwind CSS classes for styling (assumes Tailwind is set up in the project)

const sampleProducts = [
  {
    id: "p1",
    name: "Classic Sneakers",
    price: 59.99,
    desc: "Comfortable everyday sneakers.",
    img: "https://images.unsplash.com/photo-1528701800484-9b3f3e0f0b3a?w=800&q=60",
    stock: 12,
    category: "Footwear",
  },
  {
    id: "p2",
    name: "Denim Jacket",
    price: 89.0,
    desc: "Stylish denim jacket — unisex.",
    img: "https://images.unsplash.com/photo-1496128858413-b36217c2ce36?w=800&q=60",
    stock: 6,
    category: "Clothing",
  },
  {
    id: "p3",
    name: "Leather Wallet",
    price: 35.5,
    desc: "Slim leather wallet with RFID blocking.",
    img: "https://images.unsplash.com/photo-1542291026-7eec264c27ff?w=800&q=60",
    stock: 20,
    category: "Accessories",
  },
  {
    id: "p4",
    name: "Bluetooth Headphones",
    price: 129.99,
    desc: "Noise-cancelling over-ear headphones.",
    img: "https://images.unsplash.com/photo-1518444024440-6c6b6b3b5f5a?w=800&q=60",
    stock: 8,
    category: "Electronics",
  },
];

export default function EcommerceDemo() {
  const [products] = useState(sampleProducts);
  const [query, setQuery] = useState("");
  const [filter, setFilter] = useState("All");
  const [cart, setCart] = useState(() => {
    try {
      const s = localStorage.getItem("demo_cart");
      return s ? JSON.parse(s) : {};
    } catch (e) {
      return {};
    }
  });
  const [isCartOpen, setIsCartOpen] = useState(false);
  const [checkoutDone, setCheckoutDone] = useState(false);

  useEffect(() => {
    localStorage.setItem("demo_cart", JSON.stringify(cart));
  }, [cart]);

  function addToCart(productId) {
    setCart((c) => {
      const cur = { ...c };
      if (!cur[productId]) cur[productId] = 0;
      cur[productId] += 1;
      return cur;
    });
    setIsCartOpen(true);
  }

  function removeFromCart(productId) {
    setCart((c) => {
      const cur = { ...c };
      delete cur[productId];
      return cur;
    });
  }

  function changeQty(productId, delta) {
    setCart((c) => {
      const cur = { ...c };
      cur[productId] = (cur[productId] || 0) + delta;
      if (cur[productId] <= 0) delete cur[productId];
      return cur;
    });
  }

  function clearCart() {
    setCart({});
    setCheckoutDone(false);
  }

  const categories = ["All", ...Array.from(new Set(products.map((p) => p.category)))];

  const filtered = products.filter((p) => {
    const matchesQuery = p.name.toLowerCase().includes(query.toLowerCase()) || p.desc.toLowerCase().includes(query.toLowerCase());
    const matchesFilter = filter === "All" || p.category === filter;
    return matchesQuery && matchesFilter;
  });

  const cartItems = Object.entries(cart).map(([id, qty]) => {
    const product = products.find((p) => p.id === id);
    return { ...product, qty };
  });

  const subtotal = cartItems.reduce((s, it) => s + it.price * it.qty, 0);
  const shipping = subtotal > 0 ? 6.5 : 0;
  const tax = +(subtotal * 0.12).toFixed(2);
  const total = +(subtotal + shipping + tax).toFixed(2);

  function handleCheckout(e) {
    e.preventDefault();
    // Simple demo: just show success message and clear cart
    setCheckoutDone(true);
    setCart({});
    setIsCartOpen(false);
  }

  return (
    <div className="min-h-screen bg-gray-50 text-gray-800">
      <header className="bg-white shadow-sm">
        <div className="max-w-6xl mx-auto px-4 py-4 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="text-2xl font-bold">MyStore</div>
            <div className="text-sm text-gray-500">Simple e‑commerce demo</div>
          </div>

          <div className="flex items-center gap-4">
            <div className="hidden sm:flex items-center gap-2">
              <input
                value={query}
                onChange={(e) => setQuery(e.target.value)}
                placeholder="Search products..."
                className="px-3 py-2 border rounded-md text-sm focus:outline-none"
              />
              <select value={filter} onChange={(e) => setFilter(e.target.value)} className="px-2 py-2 border rounded-md text-sm">
                {categories.map((c) => (
                  <option key={c} value={c}>
                    {c}
                  </option>
                ))}
              </select>
            </div>

            <button
              onClick={() => setIsCartOpen((v) => !v)}
              className="relative bg-indigo-600 text-white px-3 py-2 rounded-md hover:bg-indigo-700"
            >
              Cart
              <span className="ml-2 bg-white text-indigo-600 px-2 py-0.5 rounded-full text-xs font-semibold">
                {Object.values(cart).reduce((a, b) => a + b, 0)}
              </span>
            </button>
          </div>
        </div>
      </header>

      <main className="max-w-6xl mx-auto px-4 py-8">
        <section className="grid grid-cols-1 lg:grid-cols-4 gap-6">
          <div className="lg:col-span-3">
            <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
              {filtered.map((p) => (
                <article key={p.id} className="bg-white rounded-2xl shadow p-4 flex flex-col">
                  <img src={p.img} alt={p.name} className="h-40 w-full object-cover rounded-lg mb-3" />
                  <div className="flex-1">
                    <h3 className="font-semibold">{p.name}</h3>
                    <p className="text-sm text-gray-500 mt-1">{p.desc}</p>
                  </div>
                  <div className="mt-4 flex items-center justify-between">
                    <div>
                      <div className="text-lg font-bold">${p.price.toFixed(2)}</div>
                      <div className="text-xs text-gray-400">Stock: {p.stock}</div>
                    </div>
                    <div className="flex items-center gap-2">
                      <button
                        onClick={() => addToCart(p.id)}
                        className="px-3 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700"
                      >
                        Add
                      </button>
                    </div>
                  </div>
                </article>
              ))}

              {filtered.length === 0 && (
                <div className="col-span-full text-center text-gray-500">No products match your search.</div>
              )}
            </div>
          </div>

          <aside className="hidden lg:block">
            <div className="sticky top-6 bg-white p-4 rounded-2xl shadow">
              <h4 className="font-semibold">Quick Filters</h4>
              <div className="mt-3 flex flex-col gap-2">
                {categories.map((c) => (
                  <button
                    key={c}
                    onClick={() => setFilter(c)}
                    className={`text-sm px-3 py-2 rounded-md text-left ${filter === c ? "bg-indigo-600 text-white" : "border text-gray-700"}`}
                  >
                    {c}
                  </button>
                ))}
              </div>

              <hr className="my-3" />
              <div>
                <h5 className="text-sm font-medium">About this demo</h5>
                <p className="text-xs text-gray-500 mt-2">This is a front-end demo. No real payments are processed.</p>
              </div>
            </div>
          </aside>
        </section>

        {/* Mobile search area */}
        <div className="sm:hidden mt-6">
          <input
            value={query}
            onChange={(e) => setQuery(e.target.value)}
            placeholder="Search products..."
            className="w-full px-3 py-2 border rounded-md text-sm"
          />
        </div>

        {/* Cart drawer (simple) */}
        <div
          className={`fixed right-0 top-0 h-full w-full sm:w-96 bg-white shadow-2xl transform transition-transform duration-300 z-50 ${
            isCartOpen ? "translate-x-0" : "translate-x-full"
          }`}
        >
          <div className="p-4 flex items-center justify-between border-b">
            <h3 className="text-lg font-semibold">Your Cart</h3>
            <div className="flex items-center gap-2">
              <button onClick={() => { clearCart(); }} className="text-sm text-red-500">Clear</button>
              <button onClick={() => setIsCartOpen(false)} className="px-3 py-1 bg-gray-100 rounded">Close</button>
            </div>
          </div>

          <div className="p-4 overflow-y-auto h-[calc(100%-220px)]">
            {cartItems.length === 0 && <div className="text-gray-500">Your cart is empty.</div>}

            {cartItems.map((it) => (
              <div key={it.id} className="flex items-center gap-3 py-3 border-b">
                <img src={it.img} alt={it.name} className="w-16 h-16 object-cover rounded" />
                <div className="flex-1">
                  <div className="font-medium">{it.name}</div>
                  <div className="text-sm text-gray-500">${it.price.toFixed(2)}</div>
                  <div className="mt-2 flex items-center gap-2">
                    <button onClick={() => changeQty(it.id, -1)} className="px-2 py-1 border rounded">-</button>
                    <div className="px-3">{it.qty}</div>
                    <button onClick={() => changeQty(it.id, +1)} className="px-2 py-1 border rounded">+</button>
                    <button onClick={() => removeFromCart(it.id)} className="ml-2 text-red-500 text-sm">Remove</button>
                  </div>
                </div>
              </div>
            ))}
          </div>

          <div className="p-4 border-t">
            <div className="flex justify-between text-sm">
              <div>Subtotal</div>
              <div>${subtotal.toFixed(2)}</div>
            </div>
            <div className="flex justify-between text-sm mt-1">
              <div>Shipping</div>
              <div>${shipping.toFixed(2)}</div>
            </div>
            <div className="flex justify-between text-sm mt-1">
              <div>Tax (12%)</div>
              <div>${tax.toFixed(2)}</div>
            </div>
            <div className="flex justify-between font-semibold mt-3 text-lg">
              <div>Total</div>
              <div>${total.toFixed(2)}</div>
            </div>

            {!checkoutDone ? (
              <form onSubmit={handleCheckout} className="mt-4">
                <h4 className="text-sm font-medium">Checkout (demo)</h4>
                <input required name="name" placeholder="Full name" className="w-full mt-2 px-3 py-2 border rounded" />
                <input required name="email" type="email" placeholder="Email" className="w-full mt-2 px-3 py-2 border rounded" />
                <div className="mt-3 flex gap-2">
                  <button disabled={cartItems.length===0} type="submit" className="flex-1 px-3 py-2 bg-green-600 text-white rounded hover:bg-green-700 disabled:opacity-60">
                    Place order
                  </button>
                  <button type="button" onClick={() => { setIsCartOpen(false); }} className="px-3 py-2 border rounded">Continue shopping</button>
                </div>
              </form>
            ) : (
              <div className="mt-4 text-center text-green-700">
                <div className="font-semibold">Thank you — order placed (demo)!</div>
                <div className="text-sm text-gray-600 mt-2">No real payments were processed.</div>
              </div>
            )}
          </div>
        </div>

      </main>

      <footer className="bg-white border-t mt-12">
        <div className="max-w-6xl mx-auto px-4 py-6 text-sm text-gray-500">© {new Date().getFullYear()} MyStore — This is a demo site.</div>
      </footer>
    </div>
  );
}
