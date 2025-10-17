```text

            <button
              key={item.id}
              onClick={() => onSelect(item.id)}
              className={`
                relative flex flex-col items-center justify-center gap-2 
                p-3 rounded-2xl transition-all duration-300
                hover:scale-[1.02] bg-white
                ${isSelected ? 'shadow-xl border-[2.5px] border-black' : 'shadow-sm hover:shadow-lg border-[1.5px] border-gray-200'}
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
                {/* Selection indicator */}
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
                className={`text-xs font-medium ${isSelected ? 'text-black' : 'text-gray-700'}`}
              >
                {colorItem.label}
              </span>
            </button>

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
                      borderImage: 'linear-gradient(135deg, #FF6B9D 0%, #C239D3 100%) 1',
                      borderImageSlice: 1
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
                className={`text-lg font-medium ${isSelected ? 'text-black' : 'text-gray-700'}`}
              >
                {item.label}
              </span>
            </button>

<button
              key={item.id}
              onClick={() => onSelect(item.id)}
              className={`
                relative flex flex-col gap-3 rounded-2xl border-3 
                overflow-hidden transition-all duration-200
                hover:scale-105 hover:shadow-lg
                ${isSelected ? 'border-black shadow-md' : 'border-gray-200 hover:border-gray-300'}
              `}
            >
              {/* Selection indicator */}
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
                    // Fallback to gradient if image fails to load
                    const target = e.target as HTMLImageElement
                    target.style.display = 'none'
                    if (target.parentElement) {
                      target.parentElement.style.background =
                        'linear-gradient(135deg, #667eea 0%, #764ba2 100%)'
                    }
                  }}
                />
              </div>

              {/* Label */}
              <div className="px-4 py-3">
                <span
                  className={`text-base font-medium ${isSelected ? 'text-black' : 'text-gray-700'}`}
                >
                  {item.label}
                </span>
              </div>
            </button>


<button
            onClick={() => navigate('/')}
            className="flex-1 py-4 rounded-xl bg-black text-white font-bold text-2xl
                       border-2 border-black
                       hover:bg-gray-800 hover:scale-[1.02] 
                       active:scale-[0.98] active:bg-gray-900
                       transition-all duration-150 ease-out
                       shadow-[4px_4px_0px_0px_rgba(0,0,0,0.3)]
                       hover:shadow-[6px_6px_0px_0px_rgba(0,0,0,0.3)]
                       active:shadow-[2px_2px_0px_0px_rgba(0,0,0,0.3)]"
          >
            처음으로
          </button>

          {/* Start Button - Green */}
          <button
            onClick={onStart}
            className="flex-1 py-4 rounded-xl bg-[#61FF96] text-black font-bold text-2xl
                       border-2 border-black
                       hover:bg-[#4ee87d] hover:scale-[1.02]
                       active:scale-[0.98] active:bg-[#3dd66a]
                       transition-all duration-150 ease-out
                       shadow-[4px_4px_0px_0px_rgba(0,0,0,0.3)]
                       hover:shadow-[6px_6px_0px_0px_rgba(0,0,0,0.3)]
                       active:shadow-[2px_2px_0px_0px_rgba(0,0,0,0.3)]"
          >
            시작
          </button>

<button
          onClick={onBack}
          className="flex-1 py-4 rounded-xl bg-black text-white font-bold text-2xl
                     border-2 border-black
                     hover:bg-gray-800 hover:scale-[1.02] 
                     active:scale-[0.98] active:bg-gray-900
                     transition-all duration-150 ease-out
                     shadow-[4px_4px_0px_0px_rgba(0,0,0,0.3)]
                     hover:shadow-[6px_6px_0px_0px_rgba(0,0,0,0.3)]
                     active:shadow-[2px_2px_0px_0px_rgba(0,0,0,0.3)]"
        >
          이전
        </button>

        {/* Next/Complete Button - Green or Gray */}
        <button
          onClick={onNext}
          disabled={!canProceed}
          className={`flex-1 py-4 rounded-xl font-bold text-2xl border-2
            ${
              canProceed
                ? `bg-[#61FF96] text-black border-black
                   hover:bg-[#4ee87d] hover:scale-[1.02]
                   active:scale-[0.98] active:bg-[#3dd66a]
                   transition-all duration-150 ease-out
                   shadow-[4px_4px_0px_0px_rgba(0,0,0,0.3)]
                   hover:shadow-[6px_6px_0px_0px_rgba(0,0,0,0.3)]
                   active:shadow-[2px_2px_0px_0px_rgba(0,0,0,0.3)]`
                : 'bg-gray-300 text-gray-500 border-gray-400 cursor-not-allowed opacity-60'
            }`}
        >
          {isLastStep ? '결과보기' : '다음'}
        </button>

<button
          onClick={() => navigate('/')}
          className="w-10 h-10 flex items-center justify-center rounded-lg hover:bg-gray-200/50 
                     transition-colors duration-150"
          aria-label="닫기"
        >
          <XIcon size={24} weight="regular" className="text-gray-600" />
        </button>

<button
              className={`text-md relative inline-block pb-3 ${
                currentTab === 'personality' ? 'font-bold text-black' : 'font-medium text-gray-400'
              }`}
            >
              Personality
              {currentTab === 'personality' && (
                <div className="absolute bottom-0 left-1/2 transform -translate-x-1/2">
                  <div className="w-0 h-0 border-l-[6px] border-r-[6px] border-t-[8px] border-l-transparent border-r-transparent border-t-black" />
                </div>
              )}
            </button>

<button
              className={`text-md relative inline-block pb-3 ${
                currentTab === 'signature' ? 'font-bold text-black' : 'font-medium text-gray-400'
              }`}
            >
              Signature
              {currentTab === 'signature' && (
                <div className="absolute bottom-0 left-1/2 transform -translate-x-1/2">
                  <div className="w-0 h-0 border-l-[6px] border-r-[6px] border-t-[8px] border-l-transparent border-r-transparent border-t-black" />
                </div>
              )}
            </button>

<button
              className={`text-md relative inline-block pb-3 ${
                currentTab === 'type' ? 'font-bold text-black' : 'font-medium text-gray-400'
              }`}
            >
              Type
              {currentTab === 'type' && (
                <div className="absolute bottom-0 left-1/2 transform -translate-x-1/2">
                  <div className="w-0 h-0 border-l-[6px] border-r-[6px] border-t-[8px] border-l-transparent border-r-transparent border-t-black" />
                </div>
              )}
            </button>

        <button
          onClick={() => navigate('/')}
          className="w-10 h-10 flex items-center justify-center rounded-lg hover:bg-gray-200/50 
                     transition-colors duration-150"
          aria-label="닫기"
        >
          <XIcon size={24} weight="regular" className="text-gray-600" />
        </button>

<button
                        onClick={() => adjustIntensity(note.id, -10)}
                        className="w-5 h-5 rounded-full bg-white border border-gray-300 hover:border-gray-400 hover:bg-gray-50
                                   flex items-center justify-center transition-colors disabled:opacity-40"
                        disabled={note.intensity === 0}
                      >
                        <MinusIcon size={10} weight="bold" className="text-gray-700" />
                      </button>
                      <button
                        onClick={() => adjustIntensity(note.id, 10)}
                        className="w-5 h-5 rounded-full bg-white border border-gray-300 hover:border-gray-400 hover:bg-gray-50
                                   flex items-center justify-center transition-colors disabled:opacity-40"
                        disabled={note.intensity === 100}
                      >
                        <PlusIcon size={10} weight="bold" className="text-gray-700" />
                      </button>

<button
            onClick={() => navigate('/custom')}
            className="flex-1 py-4 rounded-xl bg-black text-white font-bold text-2xl
                       border-2 border-black
                       hover:bg-gray-800 hover:scale-[1.02]
                       active:scale-[0.98] active:bg-gray-900
                       transition-all duration-150 ease-out
                       shadow-[4px_4px_0px_0px_rgba(0,0,0,0.3)]
                       hover:shadow-[6px_6px_0px_0px_rgba(0,0,0,0.3)]
                       active:shadow-[2px_2px_0px_0px_rgba(0,0,0,0.3)]"
          >
            다시 해보기
          </button>

          {/* Experience Button - Green */}
          <button
            onClick={() => {
              // TODO: 향수 체험하기 기능 구현
              console.log('향수 체험하기')
            }}
            className="flex-1 py-4 rounded-xl bg-[#61FF96] text-black font-bold text-2xl
                       border-2 border-black
                       hover:bg-[#4ee87d] hover:scale-[1.02]
                       active:scale-[0.98] active:bg-[#3dd66a]
                       transition-all duration-150 ease-out
                       shadow-[4px_4px_0px_0px_rgba(0,0,0,0.3)]
                       hover:shadow-[6px_6px_0px_0px_rgba(0,0,0,0.3)]
                       active:shadow-[2px_2px_0px_0px_rgba(0,0,0,0.3)]"
          >
            향수 체험하기
          </button>

        <button
          onClick={() => navigate('/')}
          className="w-10 h-10 flex items-center justify-center rounded-lg hover:bg-gray-200/50 
                     transition-colors duration-150"
          aria-label="닫기"
        >
          <XIcon size={24} weight="regular" className="text-gray-600" />
        </button>

          <button
            onClick={() => navigate('/')}
            className="flex-1 py-4 rounded-xl bg-black text-white font-bold text-2xl
                       border-2 border-black
                       hover:bg-gray-800 hover:scale-[1.02]
                       active:scale-[0.98] active:bg-gray-900
                       transition-all duration-150 ease-out
                       shadow-[4px_4px_0px_0px_rgba(0,0,0,0.3)]
                       hover:shadow-[6px_6px_0px_0px_rgba(0,0,0,0.3)]
                       active:shadow-[2px_2px_0px_0px_rgba(0,0,0,0.3)]"
          >
            처음으로
          </button>

        <button
          onClick={() => navigate('/')}
          className="w-10 h-10 flex items-center justify-center rounded-lg hover:bg-gray-200/50 
                     transition-colors duration-150"
          aria-label="닫기"
        >
          <XIcon size={24} weight="regular" className="text-gray-600" />
        </button>

          <button
            onClick={() => navigate('/')}
            className="flex-1 py-4 rounded-xl bg-black text-white font-bold text-2xl
                       border-2 border-black
                       hover:bg-gray-800 hover:scale-[1.02]
                       active:scale-[0.98] active:bg-gray-900
                       transition-all duration-150 ease-out
                       shadow-[4px_4px_0px_0px_rgba(0,0,0,0.3)]
                       hover:shadow-[6px_6px_0px_0px_rgba(0,0,0,0.3)]
                       active:shadow-[2px_2px_0px_0px_rgba(0,0,0,0.3)]"
          >
            처음으로
          </button>


```