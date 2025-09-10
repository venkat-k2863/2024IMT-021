import React, { useState, useEffect } from "react";

export default function App() {
  const API_URL = "https://dummyjson.com/recipes?limit=110&skip=0";

  const [recipes, setRecipes] = useState([]);
  const [filtered, setFiltered] = useState([]);
  const [loading, setLoading] = useState(true);
  const [search, setSearch] = useState("");
  const [ingredient, setIngredient] = useState("");
  const [modalRecipe, setModalRecipe] = useState(null);

  useEffect(() => {
    fetchAndRender();
  }, []);

  async function fetchAndRender() {
    setLoading(true);
    try {
      const res = await fetch(API_URL);
      const data = await res.json();
      const list = data.recipes ?? data;
      setRecipes(Array.isArray(list) ? list : []);
      setLoading(false);
    } catch (err) {
      console.error("Fetch error:", err);
      setRecipes([]);
      setLoading(false);
    }
  }

  useEffect(() => {
    let list = recipes;
    if (search.trim()) {
      list = list.filter(
        (r) =>
          r.name?.toLowerCase().includes(search.toLowerCase()) ||
          r.title?.toLowerCase().includes(search.toLowerCase())
      );
    }
    if (ingredient) {
      list = list.filter(
        (r) =>
          Array.isArray(r.ingredients) &&
          r.ingredients.some((i) => i.trim() === ingredient)
      );
    }
    setFiltered(list);
  }, [search, ingredient, recipes]);

  const ingredients = Array.from(
    new Set(
      recipes.flatMap((r) => (Array.isArray(r.ingredients) ? r.ingredients : []))
    )
  ).sort((a, b) => a.localeCompare(b));

  function downloadJson() {
    const data = filtered.length ? filtered : recipes;
    const blob = new Blob([JSON.stringify(data, null, 2)], {
      type: "application/json",
    });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "recipes.json";
    a.click();
    URL.revokeObjectURL(url);
  }

  return (
    <div className="max-w-6xl mx-auto p-4 text-gray-100">
      <header className="flex flex-col md:flex-row justify-between gap-4 mb-6">
        <div>
          <h1 className="text-xl font-bold">Recipe Browser — DummyJSON</h1>
          <p className="text-gray-400 text-sm">
            Loads recipes from <code>{API_URL}</code>. Search, filter, view
            details, download JSON.
          </p>
        </div>
        <div className="flex flex-wrap gap-2 items-center">
          <input
            type="search"
            placeholder="Search recipes..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="bg-gray-800 border border-gray-700 rounded-lg px-3 py-2 text-sm w-56"
          />
          <select
            value={ingredient}
            onChange={(e) => setIngredient(e.target.value)}
            className="bg-gray-800 border border-gray-700 rounded-lg px-3 py-2 text-sm"
          >
            <option value="">Filter by ingredient (all)</option>
            {ingredients.map((ing) => (
              <option key={ing} value={ing}>
                {ing.length > 28 ? ing.slice(0, 28) + "…" : ing}
              </option>
            ))}
          </select>
          <button
            onClick={downloadJson}
            className="bg-yellow-400 text-black font-bold px-3 py-2 rounded-lg"
          >
            Download JSON
          </button>
          <button
            onClick={fetchAndRender}
            className="bg-blue-500 text-white font-bold px-3 py-2 rounded-lg"
          >
            Reload
          </button>
        </div>
      </header>

      {loading ? (
        <p className="text-gray-400">Loading recipes...</p>
      ) : filtered.length === 0 ? (
        <div className="border border-dashed border-gray-700 rounded-lg p-8 text-center text-gray-400">
          No recipes found for current search/filter.
        </div>
      ) : (
        <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4">
          {filtered.map((r) => (
            <article
              key={r.id}
              className="bg-gray-900 rounded-xl p-3 flex flex-col shadow-lg"
            >
              <img
                src={
                  r.image ||
                  r.thumbnail ||
                  (r.images && r.images[0]) ||
                  `https://picsum.photos/seed/recipe${r.id}/600/400`
                }
                alt={r.name || r.title}
                className="h-40 w-full object-cover rounded-md"
              />
              <h3 className="mt-2 text-lg font-semibold truncate">
                {r.name || r.title || "Untitled"}
              </h3>
              <p className="text-sm text-gray-400">
                {r.calories
                  ? `${r.calories} kcal`
                  : r.servings
                  ? `${r.servings} servings`
                  : r.cuisine || ""}
              </p>
              <div className="flex gap-2 flex-wrap text-xs text-gray-400 mt-1">
                {(r.dishTypes || []).slice(0, 3).map((t) => (
                  <span
                    key={t}
                    className="bg-gray-800 px-2 py-1 rounded-full"
                  >
                    {t}
                  </span>
                ))}
                {(r.healthLabels || []).slice(0, 2).map((t) => (
                  <span
                    key={t}
                    className="bg-gray-800 px-2 py-1 rounded-full"
                  >
                    {t}
                  </span>
                ))}
              </div>
              <div className="flex gap-2 mt-auto pt-2">
                <button
                  onClick={() => setModalRecipe(r)}
                  className="bg-yellow-400 text-black font-bold px-3 py-1 rounded-lg text-sm"
                >
                  View
                </button>
                <button
                  onClick={() =>
                    navigator.clipboard
                      .writeText(JSON.stringify(r, null, 2))
                      .then(() => alert("Recipe JSON copied"))
                      .catch(() => alert("Copy failed"))
                  }
                  className="border border-gray-700 text-gray-400 px-3 py-1 rounded-lg text-sm"
                >
                  Copy JSON
                </button>
              </div>
            </article>
          ))}
        </div>
      )}

      <footer className="mt-6 flex justify-between text-sm text-gray-400">
        <div>{filtered.length} shown</div>
        <div>Assignment: React + Tailwind + fetch</div>
      </footer>

      {/* Modal */}
      {modalRecipe && (
        <div
          className="fixed inset-0 bg-black bg-opacity-70 flex items-center justify-center z-50"
          onClick={(e) => e.target === e.currentTarget && setModalRecipe(null)}
        >
          <div className="bg-gray-900 max-w-2xl w-full max-h-[90vh] overflow-y-auto rounded-lg p-4">
            <div className="flex justify-between items-center mb-2">
              <h2 className="text-xl font-bold">
                {modalRecipe.name || modalRecipe.title}
              </h2>
              <button
                onClick={() => setModalRecipe(null)}
                className="text-gray-400"
              >
                ✕ Close
              </button>
            </div>
            <img
              src={
                modalRecipe.image ||
                modalRecipe.thumbnail ||
                (modalRecipe.images && modalRecipe.images[0]) ||
                ""
              }
              alt="recipe"
              className="h-60 w-full object-cover rounded-md mb-3"
            />
            <p className="text-gray-400 mb-2">
              {modalRecipe.cuisine
                ? `${modalRecipe.cuisine} • ${
                    modalRecipe.prepTime
                      ? modalRecipe.prepTime + " mins"
                      : ""
                  }`
                : modalRecipe.readyInMinutes
                ? `Ready in ${modalRecipe.readyInMinutes} mins`
                : ""}
            </p>
            <h3 className="font-semibold">Ingredients</h3>
            <ul className="list-disc pl-6 mb-2">
              {Array.isArray(modalRecipe.ingredients) &&
              modalRecipe.ingredients.length ? (
                modalRecipe.ingredients.map((i, idx) => (
                  <li key={idx}>{i}</li>
                ))
              ) : (
                <li>No ingredients data</li>
              )}
            </ul>
            <h3 className="font-semibold">Instructions</h3>
            <ol className="list-decimal pl-6">
              {Array.isArray(modalRecipe.instructions) &&
              modalRecipe.instructions.length ? (
                modalRecipe.instructions.map((ins, idx) => (
                  <li key={idx}>{ins}</li>
                ))
              ) : modalRecipe.instructions &&
                typeof modalRecipe.instructions === "string" ? (
                modalRecipe.instructions
                  .split(/\n+/)
                  .map((seg, idx) => <li key={idx}>{seg}</li>)
              ) : (
                <li>No instructions provided</li>
              )}
            </ol>
          </div>
        </div>
      )}
    </div>
  );
}
