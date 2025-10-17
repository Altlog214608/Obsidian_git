```text

{/* === Card Selector Button === */}
<button
  key={item.id}
  onClick={() => onSelect(item.id)}
  className={`
    relative flex flex-col items-center justify-center gap-2 
    p-3 rounded-2xl transition-all duration-300
    hover:scale-[1.02] bg-white
    ${
      isSelected
        ? 'shadow-xl border-[2.5px] border-black'
        : 'shadow-sm hover:shadow-lg border-[1.5px] border-gray-200'
    }
  `}
>
  {/* Icon */}
  {item.icon && <div className="text-black">{item.icon}</div>}

  {/* Label */}
  <div className="flex flex-col items-center gap-1">
    <span className="text-base font-bold whitespace-nowrap text-black">
      {item.label}
    </span>

    {item.subtitle && (
      <span className="text-sm font-medium whitespace-nowrap text-gray-700">
        {item.subtitle}
      </span>
    )}
  </div>
</button>

{/* === Color Selector Button === */}
<button
  key={colorItem.id}
  onClick={() => onSelect(colorItem.id)}
  className="relative flex flex-col items-center gap-1.5 transition-all duration-200 hover:scale-105"
>
  {/* Color Circle */}
  <div
    className={`
      w-20 h-20 rounded-xl transition-all duration-200
      ${isSelected ? 'ring-[3px] ring-black ring-offset-2' : ''}
      ${isLightColor ? 'border-2 border-gray-300' : ''}
    `}
    style={{ backgroundColor: colorItem.color }}
  >
    {/* Selection Indicator */}
    {isSelected && (
      <div className="w-full h-full flex items-center justify-center">
        <div className="w-5 h-5 bg-black rounded-full flex items-center justify-center">
          <svg
            className="w-3 h-3 text-white"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
          >
            <path
              strokeLinecap="round"
              strokeLinejoin="round"
              strokeWidth={3}
              d="M5 13l4 4L19 7"
            />
          </svg>
        </div>
      </div>
    )}
  </div>

  {/* Label */}
  <span
    className={`text-xs font-medium ${
      isSelected ? 'text-black' : 'text-gray-700'
    }`}
  >
    {colorItem.label}
  </span>
</button>

{/* === Icon Selector Button === */}
<button
  key={item.id}
  onClick={() => onSelect(item.id)}
  className={`
    relative flex flex-col items-center justify-center gap-3 
    p-8 rounded-2xl transition-all duration-200
    hover:scale-[1.02]
    ${
      isSelected
        ? 'bg-white border-3 border-transparent shadow-lg'
        : 'bg-white border-3 border-transparent hover:shadow-md'
    }
  `}
  style={
    isSelected
      ? {
          background: 'white',
          borderImage:
            'linear-gradient(135deg, #FF6B9D 0%, #C239D3 100%) 1',
          borderImageSlice: 1,
        }
      : undefined
  }
>
  {/* Icon */}
  <IconComponent
    size={56}
    weight="regular"
    className={isSelected ? 'text-black' : 'text-gray-700'}
  />

  {/* Label */}
  <span
    className={`text-lg font-medium ${
      isSelected ? 'text-black' : 'text-gray-700'
    }`}
  >
    {item.label}
  </span>
</button>

{/* === Image Selector Button === */}
<button
  key={item.id}
  onClick={() => onSelect(item.id)}
  className={`
    relative flex flex-col gap-3 rounded-2xl border-3 
    overflow-hidden transition-all duration-200
    hover:scale-105 hover:shadow-lg
    ${
      isSelected
        ? 'border-black shadow-md'
        : 'border-gray-200 hover:border-gray-300'
    }
  `}
>
  {/* Selection Indicator */}
  {isSelected && (
    <div className="absolute top-3 right-3 w-8 h-8 bg-black rounded-full flex items-center justify-center z-10">
      <svg
        className="w-5 h-5 text-white"
        fill="none"
        stroke="currentColor"
        viewBox="0 0 24 24"
      >
        <path
          strokeLinecap="round"
          strokeLinejoin="round"
          strokeWidth={3}
          d="M5 13l4 4L19 7"
        />
      </svg>
    </div>
  )}

  {/* Image */}
  <div className="aspect-[3/4] bg-gray-200 relative overflow-hidden">
    <img
      src={item.imageUrl}
      alt={item.label}
      className="w-full h-full object-cover"
      onError={(e) => {
        const target = e.target as HTMLImageElement;
        target.style.display = 'none';
        if (target.parentElement) {
          target.parentElement.style.background =
            'linear-gradient(135deg, #667eea 0%, #764ba2 100%)';
        }
      }}
    />
  </div>

  {/* Label */}
  <div className="px-4 py-3">
    <span
      className={`text-base font-medium ${
        isSelected ? 'text-black' : 'text-gray-700'
      }`}
    >
      {item.label}
    </span>
  </div>
</button>


```

